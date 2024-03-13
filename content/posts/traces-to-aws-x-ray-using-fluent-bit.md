---
title: "Send distributed traces to AWS X-Ray using Fluent Bit"
date: "2024-03-13"
description: "Commonly used for logging, Fluent Bit is also capable of handling traces. Learn how Fluent Bit collects and sends OTel-compliant tracing data to AWS X-Ray."
image: "https://www.datocms-assets.com/97087/1710273253-sendingtraces_creatives_twitter.png?auto=format&fit=max&w=1200"
author: "Sharad Regoti"
canonicalUrl: "https://calyptia.com/blog/traces-to-aws-x-ray-using-fluent-bit"
herobg: "https://www.datocms-assets.com/97087/1689182792-background-fluent-bit.png"
---
*This post was [originally published on the Calyptia blog](https://calyptia.com/blog/traces-to-aws-x-ray-using-fluent-bit). 
[Calyptia](https://calyptia.com) is the primary sponsor and creator of the Fluent Bit project.*

In software development, observability allows us to understand a system from the outside 
by asking questions about the system without knowing its inner workings. Furthermore, 
it allows us to troubleshoot quickly and helps answer the question, “Why is this happening?”

To be able to ask those questions to a system, the application must be instrumented. 
That is, the application code must emit signals such as traces, metrics, and logs. 
In this post, we will focus specifically on traces.

Distributed tracing involves tracking the flow of requests through a distributed system 
and collecting telemetry data such as traces and spans to monitor the system’s 
performance and behaviour. Distributed tracing helps identify performance bottlenecks, 
optimize resource utilization, and troubleshoot issues in distributed systems.

There are many platforms that are used to monitor and analyze trace data and help 
engineers spot problems, including [Chronosphere](https://chronosphere.io/), Datadog, 
and the open source Jaeger. Today, though, we will be using AWS X-Ray, a less 
commonly used platform but a convenient one for demonstration purposes since so many 
developers have AWS accounts. 

To collect and route the traces to X-Ray we’ll be using Fluent Bit, a widely used 
open-source data collection agent, processor, and forwarder. Fluent Bit is most commonly 
used for logging, but it is also capable of handling traces and metrics, making it an 
ideal single-agent choice for any type of telemetry data. 

In this post, we'll guide you through the process of sending distributed traces to 
AWS X-Ray using Fluent Bit.

## Prerequisites

* **Docker and Docker Compose:** Installed on your local machine.
* **An AWS account**
* **AWS CLI** is a tool to manage AWS services. Install the AWS CLI by following the 
official [AWS CLI installation guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html#getting-started-install-instructions). 
After installation, configure the AWS CLI with your credentials and default region by 
running AWS configure. For detailed instructions, refer to the 
[AWS CLI configuration guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)
* **Familiarity with Fluent Bit concepts** such as inputs, outputs, parsers, and filters. 
If you’re not familiar with these concepts, 
please refer to the [**official documentation**](https://docs.fluentbit.io/manual/concepts/data-pipeline).

## Distributed tracing workflow

![Diagram illustrating how applications emit trace data which is then collected by a centralized observability data shipper that sends the data to a distributed trace storage engine](https://calyptia.com/_next/image?url=https://www.datocms-assets.com/97087/1710271503-distributed-traces-4.png&w=1920&q=75)
<caption)Instrumented applications emit trace data that is collected and processed by a 
centralized agent, when then sends to data to a backend for storage and analysis</caption>

## **Generating trace data**

In a microservices architecture, applications are instrumented using specific libraries 
to send trace data in a particular format supported by the storage engine.

[**OpenTelemetry**](https://opentelemetry.io/) (OTel) has become the standard format 
for working with telemetry data. Its open-source observability framework provides a 
standardized way to collect and transmit telemetry data such as traces, logs, and metrics 
from applications.OTel provides a common set of APIs, libraries, and tools for collecting 
and analyzing telemetry data in distributed systems.

We will be using a Python (uses Flask framework) application that we’ve instrumented 
using OpenTelemetry SDKs to generate trace data in OpenTelemetry protocol (OTLP).

We will configure Fluent Bit to receive the emitted trace data using the 
[OpenTelemetry input plugin](https://docs.fluentbit.io/manual/pipeline/inputs/opentelemetry).

**Note: For simplicity and demonstration purposes, we will be using a single service 
capable of generating a hierarchical distributed trace. But in a practical scenario, there would be multiple services instrumented to generate trace data.**

## **Storing trace data in AWS X-Ray**

AWS X-Ray accepts trace requests in the form of segment documents, which can be sent 
using two primary protocols:

1. **AWS X-Ray API (HTTP):** You can send segment documents directly to the 
[AWS X-Ray API](https://docs.aws.amazon.com/xray/latest/devguide/xray-api-sendingdata.html) 
using the [PutTraceSegments](https://docs.aws.amazon.com/xray/latest/devguide/xray-api-sendingdata.html#xray-api-segments) 
API. This is done using HTTP/1.1.
2. **Direct UDP:** You can send segment documents directly to the 
[AWS X-Ray daemon](https://docs.aws.amazon.com/xray/latest/devguide/xray-api-sendingdata.html#xray-api-segments) 
(runs aside with application) over UDP. The X-Ray daemon buffers segments in a queue and 
uploads them to X-Ray in batches.

Unfortunately, AWS X-Ray utilizes a non-standards-compliant trace ID. Since Fluent Bit 
does not support the custom X-Ray API format, it cannot send trace data directly to 
AWS X-Ray. To overcome this, we will be using the [AWS Distro for OpenTelemetry (ADOT)](https://aws-otel.github.io/docs/introduction), 
which supports OTLP input and can be used with the Fluent Bit [OpenTelemetry output plugin](https://docs.fluentbit.io/manual/pipeline/outputs/opentelemetry). 
ADOT automatically converts the compliant trace ID to the format required by AWS X-Ray.

Our architecture looks like this:

![Architectural diagram showing Fluent Bit receiving trace data from an application then sending the data to AWS Distro for OpenTelemetry which send it to AWS X-Ray](https://calyptia.com/_next/image?url=https://www.datocms-assets.com/97087/1710271782-distributed-traces-1.png&w=3840&q=75)
<caption>Fluent Bit both receives and submits OTLP but it must be converted to the bespoke 
format required by AWS X-Ray</caption>

## **Configuring Fluent Bit**

Here’s the Fluent Bit configuration that enables the depicted above:


```yaml
[SERVICE]
    flush 1
    log_level info

[INPUT]
    name opentelemetry
    host 0.0.0.0
    port 3000
    successful_response_code 200
    
[OUTPUT]
    Name             	opentelemetry
    Match            	*
    Host             	aws-adot
    Port             	4318
    traces_uri       	/v1/traces
    tls              	off
    tls.verify       	off
    add_label        	app fluent-bit
```
Breaking down the configuration above, we define one input section:

### **INPUT section**

* `name opentelemetry`: Specifies the input plugin to use, which in this case is 
`opentelemetry`. This plugin is designed to receive telemetry data (metrics, logs, and 
traces) following the OpenTelemetry format.
* `host 0.0.0.0`: This binds the input listener to all available IP addresses on 
the machine, making it accessible from other machines.
* `port 3000`: Defines the port on which Fluent Bit will listen for incoming data.
* `successful_response_code 200`: This is the HTTP response code that Fluent Bit 
will send back to the sender to indicate that the data was received successfully. 
A value of `200` corresponds to HTTP OK, meaning the request has succeeded.

### **OUTPUT section**

* `Name opentelemetry`: Specifies the output plugin to use. This indicates Fluent Bit 
will forward the processed data to another service or tool supporting OpenTelemetry data.
* `Match *`*:* This pattern matches all incoming data. In Fluent Bit, the `Match` 
directive is used to filter which data is sent to a particular output based on the tag 
associated with the data. The asterisk `*\**`is a wildcard that matches all tags.
* `Host aws-adot`: Specifies the destination host to which the data will be forwarded.
* `Port 4318`: Defines the port on the destination host where the OpenTelemetry 
collector or service is listening.
* `traces_uri /v1/traces`: Sets the specific URI endpoint where trace data should be sent. 
This is part of the OpenTelemetry specification for sending trace data.
* `tls off`: Indicates that TLS (Transport Layer Security) will not be used 
for this connection, meaning data will be sent in plaintext.
* `add_label app fluent-bit`: This adds a label to the data being sent out. 
Labels are key-value pairs. Here, `app` is the key, and `fluent-bit` is the value.

With the prerequisites now elucidated, let’s implement it in practice.

### Create Fluent Bit configuration file

Create a file called `fluent-bit.conf` with the following contents:


```yaml
[SERVICE]
    flush 1
    log_level info

[INPUT]
    name opentelemetry
    host 0.0.0.0
    port 3000
    successful_response_code 200
    
[OUTPUT]
    Name             	opentelemetry
    Match            	*
    Host             	aws-adot
    Port             	4318
    traces_uri       	/v1/traces
    tls              	off
    tls.verify       	off
    add_label        	app fluent-bit
```
## **Create OTEL Configuration**

Create a file called `otel.yaml` with the following contents and replace the key 
`<put-your-aws-region>` with your AWS region.


```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318
exporters:
  awsxray:
    region: <put-your-aws-region>
service:
  pipelines:
    traces:
      receivers:
        - otlp
      exporters:
        - awsxray
```
This configuration defines how the AWS Distro for OpenTelemetry (ADOT) Collector 
operates. It specifies the collection (receivers) of telemetry data via OpenTelemetry 
Protocol (OTLP) over gRPC and HTTP, and the export (exporters) of trace data to AWS X-Ray:

* **Receivers**: Configures the ADOT Collector to receive telemetry data.


	+ **OTLP Receiver**: Accepts data over two protocols:
	
	
		- **gRPC**: Listens on `0.0.0.0:4317` for incoming gRPC connections.
		- **HTTP**: Listens on `0.0.0.0:4318` for incoming HTTP connections.
* **Exporters**: Defines how and where processed data is sent.


	+ **AWS X-Ray Exporter**: Configured to send trace data to the AWS X-Ray service 
	in the specified AWS region.
* **Service**:


	+ **Pipelines**: Organizes the flow of data from receivers to exporters.
	
	
		- **Traces Pipeline**: Specific for trace data, it uses the `otlp` receiver to 
		collect data and the `awsxray` exporter to send the data to AWS X-Ray.

This configuration sets up the ADOT Collector to collect telemetry data using OTLP over 
both gRPC and HTTP and to export trace data to AWS X-Ray for analysis and visualization.

## Create Docker Compose Configuration

Create a file called `docker-compose.yml` with the following contents and replace 
these two keys `<put-your-aws-access-keys-id>` and `<put-your-aws-secret-access-key>` 
with your AWS credentials.


```yaml
version: '3.8'
services:
  aws-adot:
    image: public.ecr.aws/aws-observability/aws-otel-collector:latest
    container_name: aws-adot
    ports:
      - "4317:4317" # Grpc port
      - "4318:4318" # Http port
      - "55679:55679"
    volumes:
      - "./otel.yaml:/otel.yaml"
    environment:
      - AWS_REGION=ap-south-1
      - AWS_ACCESS_KEY_ID=<put-your-aws-access-keys-id>
      - AWS_SECRET_ACCESS_KEY=<put-your-aws-secret-access-key>
    command: ["--config", "/otel.yaml"]
    restart: "no"

  fluent-bit:
    image: cr.fluentbit.io/fluent/fluent-bit:2.2
    container_name: fluent-bit
    ports:
      - "3000:3000"
    volumes:
      - "./fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf"
    restart: "no"

  trace-generator:
    image: sharadregoti/trace-generator:v0.1.0
    container_name: trace-generator
    ports:
      - "5000:5000"
    environment:
      - OTEL_HOST_ADDR=fluent-bit:3000
    restart: "no"
```
This `docker-compose.yml` file defines a multi-container setup with three services: `aws-adot`, `fluent-bit`, and `trace-generator`.

* **aws-adot**:


	+ Uses the `public.ecr.aws/aws-observability/aws-otel-collector:latest` image.
	+ Exposes ports `4317` (gRPC), `4318` (HTTP).
	+ Mounts a local `otel.yaml` configuration file into the container.
	+ Configures AWS credentials and region through environment variables.
	+ Specifies a command to use the mounted config file
* **fluent-bit**:


	+ Based on the `cr.fluentbit.io/fluent/fluent-bit:2.2` image.
	+ Exposes port `3000` for log processing.
	+ Mounts a local `fluent-bit.conf` configuration file into the container.
* **trace-generator**:


	+ Uses the `sharadregoti/trace-generator:v0.1.0` image.
	+ Exposes port `5000`.
	+ Uses an environment variable to specify the `fluent-bit` service as the destination for trace data.

## Start Docker Containers

Execute the below command to start all docker containers:


```bash
docker-compose up
```
## Generate Traces by Hitting the Sample App

Open a new terminal and execute the below curl request to generate a trace:


```bash
curl -X GET http://localhost:5000/generate-hierarchical
or
curl -X GET http://localhost:5000/generate
```
## Go to the AWS Console and Open AWS X-Ray

You will observe a new trace is generated as shown in the below image.

![Screenshot of X-Ray console showing a newly generated trace](https://calyptia.com/_next/image?url=https://www.datocms-assets.com/97087/1710274188-distributed-traces-5.png&w=3840&q=75)

Click on the newly created trace to view the detailed information about the request.

![Trace detail screen in X-Ray](https://calyptia.com/_next/image?url=https://www.datocms-assets.com/97087/1710274268-distributed-traces-2.png&w=3840&q=75)

## Clean Up


```bash
# Press ctrl + c in the terminal instance where containers are running in foreground
docker-compose down
```
## **Conclusion**

In this post, we've walked through the essentials of setting up distributed tracing 
with AWS X-Ray and Fluent Bit, demonstrating how to seamlessly integrate trace data 
collection and forwarding in a microservices environment. By leveraging Docker, 
AWS X-Ray, and Fluent Bit, developers can achieve a robust observability framework 
that is both scalable and easy to implement.

## Learn more

To learn more about Fluent Bit, visit the [project website](https://fluentbit.io/) or 
the recently launched [Fluent Bit Academy](https://calyptia.com/on-demand-webinars) 
where you will find hours of on-demand training videos covering best practices and how-to’s on advanced processing, routing, and all things Fluent Bit. Here’s a sample of what you can find there:

* [Fluent Bit for Windows](https://calyptia.com/on-demand-webinars/on-demand-fluent-bit-for-windows?utm_campaign=Monthly%20Newsletter&utm_source=hs_email&utm_medium=email&_hsenc=p2ANqtz-831Ig-GQT_HdCYzE8VOFU36Qjed_MLIOgpbD7FmV_CqEG6vQdzqJpZAvBQ0Ux5-v4CqizQ)
* [Getting Started with Fluent Bit and OpenSearch](https://calyptia.com/on-demand-webinars/getting-started-with-fluent-bit-and-opensearch?utm_campaign=Monthly%20Newsletter&utm_source=hs_email&utm_medium=email&_hsenc=p2ANqtz-831Ig-GQT_HdCYzE8VOFU36Qjed_MLIOgpbD7FmV_CqEG6vQdzqJpZAvBQ0Ux5-v4CqizQ)
* [Getting Started with Fluent Bit and OpenTelemetry](https://calyptia.com/on-demand-webinars/getting-started-with-fluent-bit-and-opentelemetry?utm_campaign=Monthly%20Newsletter&utm_source=hs_email&utm_medium=email&_hsenc=p2ANqtz-831Ig-GQT_HdCYzE8VOFU36Qjed_MLIOgpbD7FmV_CqEG6vQdzqJpZAvBQ0Ux5-v4CqizQ)

We also invite you to join the vibrant Fluent community. 
Visit the [project’s GitHub repository](https://github.com/fluent/fluent-bit) to learn 
how to become a contributor. Or join the [Fluent Slack](https://launchpass.com/fluent-all) 
where you will find thousands of fellow Fluent Bit and Fluentd users helping one 
another with issues and discussing the projects’ roadmaps. 

