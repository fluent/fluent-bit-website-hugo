---
title: "Kubernetes Metadata Enrichment with Fluent Bit (with Troubleshooting Tips)"
date: "2023-11-30"
description: "Learn how to enrich Kubelet logs with metadata from the K8s API server using Fluent Bit along with troubleshooting tips for common misconfigurations."
image: "/images/blog/1701353456-general-fluent-bit-preview-card.png"
author: "Patrick Stephens"
canonicalUrl: "https://chronosphere.io/learn/fluent-bit-kubernetes-filter/"
herobg: "/images/blog/1689182792-background-fluent-bit.png"
---
*This post is [republished from the Chronosphere blog](https://chronosphere.io/learn/fluent-bit-kubernetes-filter/). With [Chronosphere’s acquisition of Calyptia](https://chronosphere.io/news/chronosphere-acquires-calyptia/) in 2024, Chronosphere became the [primary corporate sponsor of Fluent Bit](https://chronosphere.io/fluent-bit/). Eduardo Silva — the original creator of Fluent Bit and co-founder of Calyptia — leads a team of Chronosphere engineers dedicated full-time to the project, ensuring its continuous development and improvement.*

When run in Kubernetes (K8s) as a [daemonset](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/), Fluent Bit can ingest Kubelet logs and enrich them with additional metadata from the Kubernetes API server. This includes any annotations or labels on the pod and information about the namespace, pod, and the container the log is from. It is very simple to do, and, in fact, it is also the default setup when deploying Fluent Bit via the [helm chart](https://docs.fluentbit.io/manual/installation/kubernetes).

The [documentation](https://docs.fluentbit.io/manual/pipeline/filters/kubernetes) goes into the full details of what metadata are available and how to configure the Fluent Bit Kubernetes filter plugin to gather them. This post primarily gives an overview of how the filter works and provides common troubleshooting tips, particularly with issues caused by misconfiguration.

## How to get Kubernetes metadata?

Let us take a step back and look at what information is required to query the K8s API server for metadata about a particular pod. We need two things:

1. Namespace
2. Pod name

Cunningly, the Kubelet logs on the node [have to provide this information in their filename](https://github.com/kubernetes/design-proposals-archive/blob/main/node/kubelet-cri-logging.md) by design. This information enables Fluent Bit to query the K8s API server when all it has is the log file. Therefore, given a pod log file(name), we should be able to query the K8s API server for the rest of the metadata describing the pod.

## Using Fluent Bit to enrich the logs

First off, we need the actual logs from the Kubelet. This is typically done by using a daemonset to ensure a Fluent Bit pod runs on every node and then mounts the Kubelet logs from the node into the pod.

Now that we have the log files themselves we should be able to extract enough information to query the K8s API server. We do this with a default setup using the *tail* plugin to read the log files and inject the filename into the tag:


```yaml
[INPUT]
   Name tail
   Tag kube.*
   Path /var/log/containers/*.log
   multiline.parser  docker, cri
```

Wildcards in the tag are handled in a [special way](https://docs.fluentbit.io/manual/pipeline/inputs/tail#config) for the tag filter. This configuration will  inject the full path and filename for the log file into the tag after the *kube.* prefix.

Once the *kubernetes* filter receives these records, it parses the tag to extract the information required. To do so, it needs the *kube\_tag\_prefix* value to strip off any redundant tag or path to leave just the log filename with the three things required to query the K8s API server. Using the defaults would look like this:


```
[FILTER]
    Name             kubernetes
    Match            kube.*
    Kube_Tag_Prefix  kube.var.log.containers.
```
Fluent Bit inserts the extra metadata from the K8s API server under the top-level *kubernetes* key.

Using an example, we can see how this flows through the system.

Log file:

*/var/log/container/apache-logs-annotated\_default\_apache-aeeccc7a9f00f6e4e066aeff0434cf80621215071f1b20a51e8340aa7c35eac6.log*

Gives us a tag like so (slashes are replaced with dots):

*kube.var.log.containers.apache-logs-annotated\_default\_apache-aeeccc7a9f00f6e4e066aeff0434cf80621215071f1b20a51e8340aa7c35eac6.log*

We then strip of *kube\_tag\_prefix*:

*apache-logs-annotated\_default\_apache-aeeccc7a9f00f6e4e066aeff0434cf80621215071f1b20a51e8340aa7c35eac6.log*

Now we can extract the relevant fields with a regex:

*(?<pod\_name>[a-z0-9](?:[-a-z0-9]\*[a-z0-9])?(?:\.[a-z0-9]([-a-z0-9]\*[a-z0-9])?)\*)\_(?<namespace\_name>[^\_]+)\_(?<container\_name>.+)-(?<container\_id>[a-z0-9]{64})\.log$*

* pod\_name  = *apache-logs-annotated*
* namespace = *default*
* container\_name = *apache*
* container\_id = *aeeccc7a9f00f6e4e066aeff0434cf80621215071f1b20a51e8340aa7c35eac6*

![Process illustration demonstrating how the Kubernetes filter works](/images/blog/1701369935-fluent-bit-kubernetes-filter-process.png)<caption>The Fluent Bit Kubernetes filter extracts information from the log filename in order to query the K8s API server to retrieve metadata that is then added to the log file.</caption>

## Misconfiguration woes

There are a few common errors that we frequently see in community channels:

1. Mismatched configuration of tags and prefix
2. Invalid RBAC/unauthorised
3. Dangling symlinks for pod logs
4. Caching affecting dynamic labels
5. Incorrect parsers

### Mismatched tag and tag prefix

The most common problems occur when the default *tag* is changed for the *tail* input plugin or when a different path is used for the logs. When this happens, the *kube\_tag\_prefix* must also be changed to ensure it strips everything off except the filename. 

The *kubernetes* filter will otherwise end up with a garbage filename that it either complains about immediately or it injects invalid data into the request to the K8s API server. In either case, the filter will not enrich the log record as it has no additional data to add. 

Typically, you will see a warning message in the log if the tag is obviously wrong, or with *log\_level debug*, you can see the requests to the K8s API server with invalid pod name or namespace plus the response indicating there is no such pod.


```bash
$ kubectl logs fluent-bit-cs6sg
…
[2023/11/30 10:08:14] [debug] [filter:kubernetes:kubernetes.0] Send out request to API Server for pods information
[2023/11/30 10:08:14] [debug] [http_client] not using http_proxy for header
[2023/11/30 10:08:14] [debug] [http_client] server kubernetes.default.svc:443 will close connection #60
[2023/11/30 10:08:14] [debug] [filter:kubernetes:kubernetes.0] Request (ns=default, pod=s.fluent-bit-cs6sg) http_do=0, HTTP Status: 404
[2023/11/30 10:08:14] [debug] [filter:kubernetes:kubernetes.0] HTTP response
{"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"pods \"s.fluent-bit-cs6sg\" not found","reason":"NotFound","details":{"name":"s.fluent-bit-cs6sg","kind":"pods"},"code":404}
```

This example was created using a configuration file like below for the official [helm chart](https://github.com/fluent/helm-charts/tree/main/charts/fluent-bit). As you can see we have added two characters to the default tag prefix (*my*) and you can see above in the *details* for the error that the name of the pod has two extra characters in the prefix: it should be *fluent-bit-cs6sg* but is *s.fluent-bit-cs6sg*, no such pod exists so it reports a failure. Without *log\_level debug* you just get no metadata.


```yaml
config:
  service: |
    [SERVICE]
        Daemon Off
        Log_Level debug

  inputs: |
    [INPUT]
        Name tail
        Path /var/log/containers/*.log
        multiline.parser docker, cri
        Tag mykube.*
        Mem_Buf_Limit 5MB
        Skip_Long_Lines On

  filters: |
    [FILTER]
        Name kubernetes
        Match *
```
### Invalid RBAC

The Fluent Bit pod must have the relevant roles added to its service account that allow it to query the K8s API for the information it needs. Unfortunately, this error is typically just reported as a connectivity warning to the K8s API server, so it can be easily missed. 

To troubleshoot [this issue](https://github.com/fluent/fluent-bit/issues/6551), use *log\_level debug* to see the response from the K8s API server. The message will basically say “missing permissions to do X” or something similar and then it is obvious what is wrong.


```bash
[2022/12/08 15:53:38] [ info] [filter:kubernetes:kubernetes.0] testing connectivity with API server...
[2022/12/08 15:53:38] [debug] [filter:kubernetes:kubernetes.0] Send out request to API Server for pods information
[2022/12/08 15:53:38] [debug] [http_client] not using http_proxy for header
[2022/12/08 15:53:38] [debug] [http_client] server kubernetes.default.svc:443 will close connection #23
[2022/12/08 15:53:38] [debug] [filter:kubernetes:kubernetes.0] Request (ns=default, pod=calyptia-cluster-logging-316c-dcr7d) http_do=0, HTTP Status: 403
[2022/12/08 15:53:38] [debug] [filter:kubernetes:kubernetes.0] HTTP response
{"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"pods \"calyptia-cluster-logging-316c-dcr7d\" is forbidden: User \"system:serviceaccount:default:default\" cannot get resource \"pods\" in API group \"\" in the namespace \"default\"","reason":"Forbidden","details":{"name":"calyptia-cluster-logging-316c-dcr7d","kind":"pods"},"code":403}

[2022/12/08 15:53:38] [ warn] [filter:kubernetes:kubernetes.0] could not get meta for POD calyptia-cluster-logging-316c-dcr7d
```

In the example above you can see without *log\_level debug* all you will get is the warning message:


```bash
[2022/12/08 15:53:38] [ info] [filter:kubernetes:kubernetes.0] testing connectivity with API server...
[2022/12/08 15:53:38] [ warn] [filter:kubernetes:kubernetes.0] could not get meta for POD calyptia-cluster-logging-316c-dcr7d
```

### Dangling symlinks

Kubernetes has evolved over the years, and new container runtimes have also come along. As a result, the filename requirements for Kubelet logs may be handled using a symlink from a correctly named pod log file to the actual container log file created by the container runtime. When mounting the pod logs into your container, ensure they are not dangling links and that their destination is also correctly mounted.

### Caching

Fluent Bit caches the response from the K8s API server to prevent rate limiting or overloading the server. As a result, if annotations or labels are applied or removed dynamically then those changes will not be seen until the next time the cache is refreshed. A simple test is just to roll/delete the pod so a fresh one is deployed and check if it picks up the changes.

### Log file parsing

Another common misconfiguration is using custom container runtime parsers in the *tail* input. This problem is generally a legacy issue as previously there were no inbuilt CRI or docker multiline parsers. The current recommendation is always to configure the *tail* input using the provided parsers as per the [documentation](https://docs.fluentbit.io/manual/pipeline/inputs/tail#multiline-and-containers-v1.8):


```yaml
[INPUT]
    name              tail
    path              /var/log/containers/*.log
    multiline.parser  docker, cri
```

Do not use your own CRI or docker parsers, as they must cope with merging partial lines (identified with a *P* instead of an *F*). 

The parsers for the *tail* plugin are not applied sequentially but are mutually exclusive, with the first one matching being applied. The goal is to handle multiline logs created by the Kubelet itself. Later, you can have another filter to handle multiline parsing of the application logs themselves after they have been reconstructed here.

## What’s Next?

* **Join the Fluent Slack Community:** To learn more about Fluent Bit, we recommend joining the [Fluent Community Slack channel](https://launchpass.com/fluent-all) where you will find thousands of other Fluent Bit users.
* **Check Out Fluent & Calyptia On-Demand Webinars:** Our free [on-demand webinars](https://calyptia.com/events) that range from introductory sessions to deep dives into advanced features.
* **Learn about Calyptia Core:** Finally, you may also be interested in learning about our enterprise offering, [Calyptia Core](https://calyptia.com/products/calyptia-core), which simplifies the creation and management of telemetry data pipelines at scale. Request a [customized demo](https://calyptia.com/demo) or sign up for a [free trial](https://calyptia.com/free-trial) now.
