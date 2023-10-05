---
title: "Dynamic Log Routing Based On Kubernetes Labels using Fluent Bit & WASM"
date: "2023-10-05"
description: "Use Fluent Bit's WASM plugin to process and evaluate Kubernetes
labels in streaming logs and determine where the log data should be routed for
storage."
image: "https://www.datocms-assets.com/97087/1696524913-dynamic-routing-wasm-social.png?auto=format&fit=max&w=1200"
author: "Sharad Regoti"
canonicalUrl: "https://calyptia.com/blog/dynamic-log-routing-based-on-kubernetes-labels-using-fluent-bit-wasm"
herobg: "https://www.datocms-assets.com/97087/1689182792-background-fluent-bit.png"
---
*This post was [originally published on the Calyptia blog](https://calyptia.com/blog/dynamic-log-routing-based-on-kubernetes-labels-using-fluent-bit-wasm). 
[Calyptia](https://calyptia.com) is the primary sponsor and creator of the Fluent Bit project.*

Fluent Bit is a widely-used open-source data collection agent, processor, and forwarder 
that enables you to collect logs, metrics, and traces from various sources, filter and 
transform them, and then forward them to multiple destinations. With over 
[ten billion Docker](https://calyptia.com/blog/fluent-bit-surpasses-10-billion-docker-pulls) 
pulls, Fluent Bit has established itself as a preferred choice for log processing, 
collecting, and shipping.

Fluent Bit stands out due to its lightweight nature and flexibility, offering a
variety of inputs, outputs, and filters. This pluggable architecture allows
users to tailor Fluent Bit according to their specific logging requirements,
whether it's collecting logs from different sources, transforming log data, or
shipping them to diverse destinations.

WebAssembly (WASM) is a low-level, binary instruction format designed to be a
portable target for the compilation of high-level languages like C, C++, Golang,
and others, enabling deployment on the web and other environments. Its
portability, language flexibility, and near-native execution speed make it a
popular choice for developers.

In this post we’ll cover how WASM can be used with Fluent Bit to extend Fluent
Bit’s capabilities, enabling users to implement custom logic and
functionalities. With the integration of WASM, Fluent Bit can address a range of
unique and sophisticated use cases.

Below, we will delve into one such use case of dynamic log routing based on
Kubernetes labels.

## **Prerequisites**

* **Kubernetes Cluster:** We will deploy Fluent Bit in a Kubernetes cluster and
ship logs of application containers inside Kubernetes. We will be using an EKS
cluster, but any cluster will suffice.
* **Kubectl and Helm CLI:** Installed on your local machine.
* **Golang (1.17 / 1.18):** WASM plugins will be written using Golang.
* **Tinygo (v0.24.0 or later):** For building WASM programs.
* **Familiarity with Fluent Bit concepts:** Such as, inputs, outputs, parsers, and filters. 
If you’re unfamiliar with these concepts, please refer to the 
[official documentation](https://docs.fluentbit.io/manual/concepts/data-pipeline).

## **Understanding the Use Case**

In enterprise environments there often arises a need to route application logs to multiple 
destinations based on some criteria. With Fluent Bit, 
[routing of logs](https://docs.fluentbit.io/manual/concepts/key-concepts#tag) is achieved 
by configuring `tag` and `match` fields for each plugin.

For routing logs to multiple destinations based on some criteria, we need to
configure the following

1. An input plugin to read logs from all application containers with a generic
source `tag`.
2. Multiple output plugins (sending logs to different destinations) each with a
specific `match` value.
3. A filtering or transformation mechanism that examines the log record and
modifies the source tag with an appropriate destination tag.
4. A filter to add Kubernetes metadata to each log record. This metadata will be
used to evaluate the destination tag.

In Fluent Bit, this filtering mechanism can be performed using the 
[`Rewrite Tag`](https://docs.fluentbit.io/manual/pipeline/filters/rewrite-tag) filter, 
which evaluates a log record against a regular expression. If the expression passes, 
then the source tag is modified.

But this filter operates only with regular expressions. This restricts us to
define more intricate conditions based purely on regex. To illustrate, consider
a Kubernetes Pod having the labels below:


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-sample-pod
  labels:
    region: "ap-south-1"
    tenant: "private-a1"
    has-sensitive-data: "true"
...
```
We can have a criteria such as if an application has label `tenant` equal to
`private-a1` and `region` equal to `ap-south-1` and `has-sensitive-data` equal
to `true` then route these logs to s3 in ap-south-1 region.

Evaluating these conditions using regex would be extremely difficult.

However, with the [Fluent Bit WASM plugin](https://docs.fluentbit.io/manual/pipeline/filters/wasm), 
we can create a filter that facilitates retagging under multiple conditions. 
These conditions can incorporate logical and arithmetic operations, spanning several 
expressions — essentially, the capabilities we harness in conventional programming.

With the use case explained, let's start by writing the WASM program.

## Writing the WASM Program

Fluent Bit has no additional requirements for executing WASM plugins. We just
need to write a program in a language that can compile to WASM. We’ll be using
Golang, but it could just as easily be done in any other language that WASM
supports.

Here is our WASM filter written in Golang that supports our use case.


```go
package main

import (
  "fmt"
  "unsafe"

  "github.com/valyala/fastjson"
)

//export go_filter
func go_filter(tag *uint8, tag_len uint, time_sec uint, time_nsec uint, record *uint8, record_len uint) *uint8 {
  // btag := unsafe.Slice(tag, tag_len) // Note, requires Go 1.17 (tinygo 0.20)
  // now := time.Unix(int64(time_sec), int64(time_nsec))
  brecord := unsafe.Slice(record, record_len)

  br := string(brecord)

  var p fastjson.Parser
  value, err := p.Parse(br)
  if err != nil {
    fmt.Println(err)
    return nil
  }
  obj, err := value.Object()
  if err != nil {
    fmt.Println(err)
    return nil
  }

  kubernetesObj := obj.Get("kubernetes")
  if kubernetesObj == nil {
    s := obj.String()
    s += string(rune(0)) // Note: explicit null terminator.
    rv := []byte(s)

    return &rv[0]
  }

  labelsObj := kubernetesObj.GetObject("labels")
  if labelsObj == nil {
    s := obj.String()
    s += string(rune(0)) // Note: explicit null terminator.
    rv := []byte(s)

    return &rv[0]
  }

  var ar fastjson.Arena
  var region, tenant, hasSensitiveData, newTag string

  if labelsObj.Get("region") != nil {
    region = labelsObj.Get("region").String()
  }
  if labelsObj.Get("tenant") != nil {
    tenant = labelsObj.Get("tenant").String()
  }
  if labelsObj.Get("has-sensitive-data") != nil {
    hasSensitiveData = labelsObj.Get("has-sensitive-data").String()
  }

  if region == "\"ap-south-1\"" && tenant == "\"private-a1\"" {
    newTag = "private-a1-elastic-ap-south-1"
    if hasSensitiveData == "\"true\"" {
      newTag = "private-a1-s3-ap-south-1"
    }
  } else if region == "\"ap-south-1\"" {
    newTag = "general-elastic-ap-south-1"
    if hasSensitiveData == "\"true\"" {
      newTag = "general-s3-ap-south-1"
    }
  } else {
    newTag = "general-elastic"
    if hasSensitiveData == "\"true\"" {
      newTag = "general-s3"
    }
  }

  obj.Set("new_tag", ar.NewString(newTag))
  s := obj.String()
  s += string(rune(0)) // Note: explicit null terminator.
  rv := []byte(s)

  return &rv[0]
}

func main() {}
```
## **Program Explanation**

The core logic is written in the function `go_filter`. This function name will
also be used during WASM plugin configuration.

The WASM plugin must have the following function signature:


```go
//export go_filter
func go_filter(tag *uint8, tag_len uint, time_sec uint, time_nsec uint, record *uint8, record_len uint) *uint8
```
**Note:** The comment `//export go_filter` on function is required and it should
be the same as the function name.

Using the function parameters we will have access to the original `log record`,
`tag`, and `timestamp`. Here is an example log record:


```json
{
    "log": "2023-10-02T06:52:52.843524746Z stdout F 122.30.117.241 - - [02/Oct/2023:06:52:23 +0000] GET /vortals HTTP/1.0 204 12615",
    "kubernetes": {
        "pod_name": "app-3-5f8ff8f8c6-xv5lw",
        "namespace_name": "default",
        "pod_id": "afddd062-c1d9-4333-b3d5-36abd61990eb",
        "labels": {
            "app": "app-3",
            "pod-template-hash": "5f8ff8f8c6"
        },
        "host": "ip-172-16-18-216.ap-south-1.compute.internal",
        "container_name": "app-3",
        "docker_id": "f8236c0b81a8926343d982cf94422d24fe882a4b19ec3db2b03482b1d30c8055",
        "container_hash": "docker.io/mingrammer/flog@sha256:44180f8610fab7d4c29ff233a79e19cf28bd425c1737aa59c72c1f66613fdf41",
        "container_image": "docker.io/mingrammer/flog:0.4.3"
    }
}
```
The WASM plugin only allows modifying the log record. However, to support our
use case we have to modify the source tag based on some criteria. Therefore,
instead of modifying the tag via the WASM plugin, we'll evaluate the tag value
and insert a `new\_tag` field in the original log. Using this `new_tag`, we
utilize the `Rewrite Tag` filter for retagging based on simple equality checks.

### **Processing the Record**:

The function parameter `record` is of type byte slice—which presumably contains
a JSON string—is converted to a Go string.

This string is then parsed using the `fastjson` package.

### **Kubernetes and Labels**:

The function checks if the parsed JSON has a `kubernetes` key. If not, the
original JSON is returned.

If the `kubernetes` key exists, then it checks for a nested `labels` key.

### **Check and Assign Labels**:

It looks for specific labels like `region`, `tenant`, and `has-sensitive-data`
within the `labels` object.

Depending on the values of these labels, it determines a `newTag`.

### **Determine** `**newTag**`:

If the `region` is `ap-south-1` and the `tenant` is `private-a1`, it sets
`newTag` to `private-a1-elastic-ap-south-1`. However, if the
`has-sensitive-data` label is set to `true`, `newTag` will instead be
`private-a1-s3-ap-south-1`.

If only the `region` is `ap-south-1` without the specific `tenant` value, the
logic is similar but uses the "general" prefix.

For any other region, the default is `general-elastic`, unless
`has-sensitive-data` is `true`, in which case it becomes `general-s3`.

### **Modify and Return:**

The determined `newTag` is added to the original JSON. The modified record will
look like this:


```
{
    "log": "2023-10-02T06:52:52.843524746Z stdout F 122.30.117.241 - - [02/Oct/2023:06:52:23 +0000] GET /vortals HTTP/1.0 204 12615",
    "kubernetes": {
        "pod_name": "app-3-5f8ff8f8c6-xv5lw",
        "namespace_name": "default",
        "pod_id": "afddd062-c1d9-4333-b3d5-36abd61990eb",
        "labels": {
            "app": "app-3",
            "pod-template-hash": "5f8ff8f8c6"
        },
        "host": "ip-172-16-18-216.ap-south-1.compute.internal",
        "container_name": "app-3",
        "docker_id": "f8236c0b81a8926343d982cf94422d24fe882a4b19ec3db2b03482b1d30c8055",
        "container_hash": "docker.io/mingrammer/flog@sha256:44180f8610fab7d4c29ff233a79e19cf28bd425c1737aa59c72c1f66613fdf41",
        "container_image": "docker.io/mingrammer/flog:0.4.3"
    },
    // 👇 New field added by WASM plugin
    "new_tag": "general-elastic"
}
```
The function then converts the modified JSON string back to a byte slice and
returns a pointer to its first byte.

Note that there's an explicit null terminator added to the end of the string
before converting it back to a byte slice. This is necessary for compatibility
with whatever system reads this output, perhaps a C/C++ framework.

The`main` function is empty because the primary function here (`go_filter`) is
meant to be exported and used as a plugin.

For more info on writing WASM plugins, follow the 
[official documentation](https://docs.fluentbit.io/manual/development/wasm-filter-plugins).

## **Instructions for Compiling the WASM Program:**

Initialize a new Golang project using the below command:


```bash
mkdir go-filter && go mod init go-filter
```
Copy the above Golang program in a file called `filter.go`.

With our filter program written, let’s compile it using `tinygo`:


```bash
tinygo build -wasm-abi=generic -target=wasi -o filter.wasm filter.go
```
## **Configuring Fluent Bit To Use WASM Plugin**

![Illustration of pipeline](https://calyptia.com/_next/image?url=https://www.datocms-assets.com/97087/1696534425-dynamic-routing-wasm-pipeline.png&w=3840&q=75)Here’s the Fluent Bit configuration that enables the log processing pipeline depicted above:


```yaml
[INPUT]
    Name  tail
    Tag   kube.*
    Path  /var/log/containers/*.log
    Exclude_Path  /var/log/containers/*default_fluent-bit*

[FILTER]
    Name   kubernetes
    Match  kube.*

[FILTER]
    Name wasm
    match kube.*
    WASM_Path /fluent-bit/etc/filter.wasm
    Function_Name go_filter

[FILTER]
    Name   rewrite_tag
    Match  kube.*
    Rule   $new_tag private-a1-s3-ap-south-1 private-a1-s3-ap-south-1 false
    Rule   $new_tag private-a1-elastic-ap-south-1 private-a1-elastic-ap-south-1 false
    Rule   $new_tag general-s3-ap-south-1 general-s3-ap-south-1 false
    Rule   $new_tag general-elastic-ap-south-1 general-elastic-ap-south-1 false
    Rule   $new_tag general-s3 general-s3 false
    Rule   $new_tag general-elastic general-elastic false

[OUTPUT]
    Name  stdout
    Match kube.*

[OUTPUT]
    Name  stdout
    Match private-a1-s3-ap-south-1

[OUTPUT]
    Name  stdout
    Match private-a1-elastic-ap-south-1

[OUTPUT]
    Name  stdout
    Match general-s3-ap-south-1

[OUTPUT]
    Name  stdout
    Match general-elastic-ap-south-1

[OUTPUT]
    Name  stdout
    Match general-s3

[OUTPUT]
    Name  stdout
    Match general-elastic
```
Breaking down the configuration above, we define one input section:

* **Tail**: This input section captures all container logs and tags them with
`kube.*`.

The filter section applies three filters:

1. **Kubernetes Filter:** This filter appends Kubernetes metadata to all logs
aligned with the `kube.*` tag.
2. **Custom WASM Filter:** This section selects all the logs that match the tag
`kube.*` and appends `new_tag` to it, whose value is evaluated as per the
criterias we discussed above.
3. **Rewrite Tag Filter:** This section selects all the logs that match the tag
`kube.*` and applies a processing rule to them. The configuration value of
`Rule` field is mapped to the format `$KEY REGEX NEW_TAG KEEP`


1. **$KEY:** The key represents the name of the *record key* that holds the
*value* that we want to use to match our regular expression. In our case, it is
`new_tag`.
2. **Regex:** We use simple equality regular expression. As WASM plugin already
evaluated the conditions.
3. **New Tag:** If our regular expression matches the value of the defined key
in the rule, we apply a new Tag for that specific record
4. **Keep:** If a rule matches, the filter emits a copy of the record with the
newly defined Tag. The `keep` property takes a boolean value to determine
whether the original record with the old Tag should be preserved and continue in
the pipeline or be discarded. In our case, we will be setting it to `false`

For more information about  the `rewrite_tag` plugin, check the 
[official documentation](https://docs.fluentbit.io/manual/pipeline/filters/log_to_metrics).

The output section of the configuration identifies multiple destinations.

We have configured seven plugins, each with a different `match` value. All of
them use the `stdout` plugin, which sends data on Fluent Bit’s standard output.
This is done for demonstration purposes only—in a practical scenario we would
have sent it to S3, Elasticsearch, or some other destination.

## **Instructions For Configuring Fluent Bit:**

### **Creating Kubernetes Configmap:**

For Fluent Bit to access the WASM plugin, the plugin should be available as a
file in the container. We will create a Kubernetes configmap which contains the
required WASM file and mount it to the Fluent Bit container as a file. Go to the
directory where `filter.wasm` exists and execute the below command:


```bash
kubectl create configmap wasm-filter --from-file=filter.wasm  --namespace=default
```
### Add Fluent Bit Helm Repo:

Use the command below to add the Fluent Bit Helm repository:


```bash
helm repo add fluent <https://fluent.github.io/helm-charts>
```
### **Override Default Configuration:**

Create a file called values.yaml with the following contents:


```yaml
config:
    inputs: |
        [INPUT]
            Name  tail
            Tag   kube.*
            Path  /var/log/containers/*.log
            Exclude_Path  /var/log/containers/*default_fluent-bit*

    filters: |
        [FILTER]
            Name   kubernetes
            Match  kube.*
        [FILTER]
            Name wasm
            match kube.*
            WASM_Path /fluent-bit/etc/wasm/config/filter.wasm
            Function_Name go_filter
        [FILTER]
            Name   rewrite_tag
            Match  kube.*
            Rule   $new_tag private-a1-s3-ap-south-1 private-a1-s3-ap-south-1 false
            Rule   $new_tag private-a1-elastic-ap-south-1 private-a1-elastic-ap-south-1 false
            Rule   $new_tag general-s3-ap-south-1 general-s3-ap-south-1 false
            Rule   $new_tag general-elastic-ap-south-1 general-elastic-ap-south-1 false
            Rule   $new_tag general-s3 general-s3 false
            Rule   $new_tag general-elastic general-elastic false

    outputs: |
        [OUTPUT]
            Name  stdout
            Match kube.*
        [OUTPUT]
            Name  stdout
            Match private-a1-s3-ap-south-1
        [OUTPUT]
            Name  stdout
            Match private-a1-elastic-ap-south-1
        [OUTPUT]
            Name  stdout
            Match general-s3-ap-south-1
        [OUTPUT]
            Name  stdout
            Match general-elastic-ap-south-1
        [OUTPUT]
            Name  stdout
            Match general-s3
        [OUTPUT]
            Name  stdout
            Match general-elastic
extraVolumes:
    - name: wasm-filter
      configMap:
          name: wasm-filter
extraVolumeMounts:
    - name: wasm-filter
      mountPath: /fluent-bit/etc/wasm/config
```
### Deploy Fluent Bit

Use the command below:


```bash
helm upgrade -i fluent-bit fluent/fluent-bit --values values.yaml
```
### Wait for Fluent Bit Pods to Run

Ensure that the Fluent Bit pods reach the `Running` state.


```
kubectl get pods
```
### **Verify Fluent Bit Logs**

Use the command below to check Fluent Bit logs


```
kubectl logs <fluent-bit-pod-name> -f
```
![screenshot of log output with our modified tag highlighted](https://calyptia.com/_next/image?url=https://www.datocms-assets.com/97087/1696532343-dynamic-routing-final-logs.png&w=3840&q=75)You should be able to view modified tags as shown in the above image.

## **Conclusion**

In this post we looked at how to use Fluent Bit and WASM to do dynamic log
routing using Kubernetes labels.

This demonstrated how Fluent Bit and WASM enable you to apply complex business
logic to streaming log data in order to meet business needs. We encourage you to
try Fluent Bit plus WASM today.

## Next steps: Learn more about Fluent Bit and data processing

The WASM plugin is just one option for processing data with Fluent Bit. If you
are interested in exploring more about Fluent Bit’s ability to process and
transform streaming data we recommend the following:

* “[Fluent Bit: Advanced Processing](https://www.youtube.com/watch?v=-gQ1111ONhU)”—this 
on-demand webinar provides an introduction to processing with Fluent Bit and demonstrates 
best practices and real-world examples for redaction, reduction, enrichment, and tagging of log data.
* “[Creating custom processing rules for Fluent Bit with Lua](https://calyptia.com/blog/creating-custom-processing-rules-for-fluent-bit-with-lua)”
—In addition to support for WASM, Fluent Bit also supports custom scripts written in Lua. This step-by-step tutorial walks you through several examples.

You may also be interested in exploring Calyptia Core, our telemetry pipeline manager that 
includes more than 20 built-in processing transformations all configurable with a few clicks. 
Learn more about Calyptia Core and [get started with a free trial](https://calyptia.com/).

