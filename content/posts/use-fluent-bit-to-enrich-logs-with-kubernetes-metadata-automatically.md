---
title: "Use Fluent Bit to Enrich Logs with Kubernetes Metadata, Automatically"
date: "2023-08-08"
description: "Learn how Kubernetes metadata can enhance traceability and enrich
diagnostics and how Fluent Bit makes it possible"
image: "/images/blog/1691520029-banner.png"
author: "Celalettin Calis"
canonicalUrl: "https://chronosphere.io/learn/enrich-logs-with-kubernetes-metadata-fluent-bit/"
herobg: "/images/background-fluent-bit.png"
---
**This post is [republished from the Chronosphere blog](https://chronosphere.io/learn/enrich-logs-with-kubernetes-metadata-fluent-bit/). 
With [Chronosphere’s acquisition of Calyptia](https://chronosphere.io/news/chronosphere-acquires-calyptia/) in 2024, Chronosphere became the [primary corporate sponsor of Fluent Bit](https://chronosphere.io/fluent-bit/)</a>. Eduardo Silva — the original creator of Fluent Bit and co-founder of Calyptia — leads a team of Chronosphere engineers dedicated full-time to the project, ensuring its continuous development and improvement.*

If you’re in the world of application orchestration (microservices, CI/CD,
multi-cloud deployments, data pipelines), Kubernetes is likely an essential tool
in your tech stack. This robust platform provides new ways for application
deployment and scaling and has revolutionized how we manage and scale
containerized apps.

Kubernetes may be your best buddy on most days, but it also can produce an
overwhelming amount of event data. This can make it difficult to pinpoint
issues, especially when you’ve started getting alerts for your workloads,
support tickets, and endless notifications on Slack from the stakeholders about
service degradation running on Kubernetes. That’s where logs come in, helping
you to identify your system’s point of failure.

If you are using K8s on a public cloud provider odds are that you are already
running Fluent Bit, an open source high-performance data collection agent, by
default. In many cases the out-of-the-box Fluent Bit instance is configured to
route data to the public cloud providers’ backend (e.g. CloudWatch, Azure
Monitor Logs, or Google Cloud Logging). What a lot of developers don’t know is
that you can also customize Fluent Bit to enrich logs with metadata and route
them to multiple locations.

With the blog below you will learn how you can take advantage of the same high
performance and low resource utilization with rich metadata enrichment and
leverage any of the 100+ backends supported by Fluent Bit.

Today we are going to talk about the value of Kubernetes metadata and how using
this with Fluent Bit can enable you to enhance traceability and enrich
diagnostics.

![Fluent Bit + Kubernetes logging](/images/blog/1691510930-fluent-bit-kubernetes-logging.png)

## Understanding Kubernetes Metadata

Metadata, in its simplest form, is the “info about the info.” It’s like looking
up your favorite movie on IMDB — it tells you who’s in it, who directed it, how
long it is, and so on.

In the context of Kubernetes, metadata contains the details of your running
workloads. It covers names, namespaces, labels, annotations, and more.

To illustrate, let’s consider a simple example. Suppose you have a pod named
“checkout-service-2” in the “checkout” namespace. This pod might have a label
like “app=checkout-service” and an annotation like “product-owner=J. Doe.” This
information might seem basic, but this information can be a lifesaver during a
disaster. Later in this blog, we'll share some valuable tips and best practices
for creating and managing good metadata, aiming to optimize your operations.

Let’s dig a little deeper.

## How Metadata can reduce MTTR (Mean Time to Repair)

When your workloads generate extensive logs without the right tools, it can
become quite challenging to manage the data. This is where metadata becomes
invaluable.

Let’s say you have received an alert about higher-than-normal response times in
your e-commerce application. Naturally, you’d begin to check your logs. Since
your e-commerce application runs on multiple pods, all logging simultaneously,
it’s not easy to isolate the problem.

This is where the Kubernetes metadata can help. By sorting sample logs by
response time, you are able to determine that the endpoints are related to
checkout service. The “app=checkout-service” label on the pod will enable you to
quickly isolate logs associated with this particular service in your Kubernetes
cluster.

The Kubernetes metadata will enable you to dig even deeper. By looking at the
metadata, you are able to determine that each slow request is associated with a
particular pod, “checkout-service-2.” Now that you have narrowed down the issue
to a specific pod, you uncover that this pod had been scheduled to a node in a
bad state. A quick reschedule later, your e-commerce application is back to
running properly.

Using metadata to navigate your logs will help your operations to become more
efficient and streamlined, reducing downtime and minimizing disruption to the
user experience. This proactive approach will enable you to anticipate issues
before they become significant problems, further increasing your manager's
confidence in your abilities and the robustness of the system.

## Automating Kubernetes Metadata with Fluent Bit

Fluent Bit, the lightweight and highly performant log processor and forwarder,
excels when dealing with Kubernetes metadata. Fluent Bit is designed to
automatically enrich your logs with Kubernetes metadata, tagging each log entry
with helpful information such as the pod name, namespace, labels, and more.

In addition to capturing and enriching logs, Fluent will ship your logs, fully
loaded with metadata, to your preferred log analysis platform. Fluent Bit has
more than 100 built-in integrations, including Elasticsearch, AWS Cloudwatch,
and more.

In the context of Fluent Bit, filters are components that allow you to
manipulate and process the data logs before they are sent to output. Fluent Bit
Kubernetes Filter allows you to enrich your log files with Kubernetes metadata.

After adding Kubernetes Filter block to your Fluent Bit configuration, this
filter aims to perform the following operations:

* Analyze the Tag and extract the following metadata:
+ Pod Name
+ Namespace
+ Container Name
+ Container ID
* Query Kubernetes API Server to obtain extra metadata for the POD in question:
+ Pod ID
+ Labels
+ Annotations

The data is cached locally in memory and appended to each record.

You may find the example configuration below:


```yaml
[INPUT]
    Name            tail
    Tag             kube.*
    Path            var/log/containers/*.log
    Parser          docker

[FILTER]
    Name            kubernetes
    Match           kube.*
    Kube_URL        https://kubernetes.default.svc:443
    Kube_CA_File    /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    Kube_Token_File var/run/secrets/kubernetes.io/serviceaccount/token
    Kube_Tag_Prefix kube.var.log.containers.
    Merge_Log       On
    Merge_Log_Key   log_processed
```
## Metadata Best Practices: Labels & Annotations

Now that you’ve seen the importance of Kubernetes metadata in action, let’s
share some best practices.

Use labels to categorize your Kubernetes objects. Some examples of label best
practices include:

1. Create a label to group all the components of a specific application running
across different microservices.
2. Create a label to distinguish the environment of particular workloads.

Annotations, meanwhile, are like Post-it notes on your workloads. They’re
excellent for storing extra details like a description of the workload, the
person in charge, etc. In essence, labels are for identifying, and annotations
provide additional context.

## Conclusion

We’ve discussed Kubernetes metadata and how it brings value to your logging
process. We’ve explored how it turns the challenging task of troubleshooting
into a straightforward process. Additionally, we’ve seen how Fluent Bit
automates the enrichment of your logs with metadata and ships them to your
preferred log analysis platform, such as Datadog, Splunk or AWS S3.

Just remember, as powerful as Kubernetes metadata is, it’s only as good as your
usage. Use labels and annotations effectively. Enrich your logs with as much
context as possible. Make certain that your tooling preserves the information as
the logs are routed downstream. And above all, embrace the metadata.

## Next Steps

If you are new to Fluent Bit, we provide [free hands-on learning labs](https://info.calyptia.com/learning-labs) 
using ephemeral sandbox environments. For more advanced users, we recommend our 
[Fluent Bit Summer webinar series](https://calyptia.com/blog/fluent-bit-summer-webinar-series) 
with topics such as advanced processing and routing to help you optimize your Fluent Bit usage. 
All webinars are available on-demand following the live presentation.

Also, look for our upcoming training sessions starting in September.
