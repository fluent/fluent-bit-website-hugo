---
title: "Announcing Fluent Bit v3"
date: "2024-03-19"
description: "The new release allows the filtering of Windows and MacOS metrics, 
supports SQL for parsing logs, adds support for HTTP/2, and more."
image: "../images/blog/fluent-bit-v3.png"
author: "Calyptia Team"
canonicalUrl: "https://calyptia.com/blog/fluent-bit-v3"
herobg: "/images/blog/1689182792-background-fluent-bit.png"
slug: fluent-bit-v3
aliases:
- /blog/fluent-bit-v3
---
**This post is [republished from the Chronosphere blog](https://calyptia.com/blog/fluent-bit-v3). 
With [Chronosphere’s acquisition of Calyptia](https://chronosphere.io/news/chronosphere-acquires-calyptia/) in 2024, Chronosphere became the [primary corporate sponsor of Fluent Bit](https://chronosphere.io/fluent-bit/)</a>. Eduardo Silva — the original creator of Fluent Bit and co-founder of Calyptia — leads a team of Chronosphere engineers dedicated full-time to the project, ensuring its continuous development and improvement.*

![Fluent Bit v3](/images/blog/fluent-bit-v3-1024x535.png)

# Fluent Bit v3 gives users greater control of their data and telemetry pipelines

This week at KubeCon + CloudNativeCon EU in Paris, we are announcing the release of 
Fluent Bit v3, which includes several new features as well as performance enhancements. 
The release adds support for:

* New filters for Windows and MacOS metrics.
* New SQL-based parser for searching and transforming logs in stream before routing 
them for storage and analysis.
* New and emerging standards such as HTTP/2.

A huge thank you to GitHub, SAP, IBM, Microsoft, Amazon, Google, and other maintainers 
for their contributions to the Fluent Bit v3 release. We are excited to continue 
the vision of building the most performant and versatile open source agent for collecting, 
processing, transforming, and routing telemetry data. 

![We are celebrating 13 billion downloads](/images/blog/1710511305-13billion-creatives-twitter.png)

Earlier this month, Fluent Bit surpassed 13 billion downloads from DockerHub — 
a massive accomplishment demonstrating the acceleration in adoption since it hit one 
billion just two years ago. Fluent Bit is embedded in major Kubernetes distributions, 
including those from AWS, GCP, and Azure.

Now, let’s explore some of the new features in more detail.

## Metrics filtering gives devops teams more control over their data

In 2022, Fluent Bit added metrics compatibility, including the ability to scrape 
Prometheus metrics, collect metrics from a host, and route the metrics in either 
Prometheus or OpenTelemetry format. Since then, we’ve seen wide adoption as companies 
look to centralize on a single agent for logs and metrics collection at the edge. 

With Fluent Bit v3, users can now filter and process metrics before routing them to 
their OpenTelemetry or Prometheus-based endpoints, giving users greater control over 
which metrics are prioritized for reporting. 

Fluent Bit is known for its powerful ability to process and filter logs in stream, 
including enrichment, transformation, and reduction, such as removing unnecessary 
information. With Fluent Bit v3, users can apply aspects of that same filtering and 
processing to their metrics. 

Collecting and processing Windows and MacOS metrics is a major pain point for developers, 
especially when dealing with these metrics at scale. Fluent Bit v3 helps address 
this pain by reducing the toolset and the complexity involved. With Fluent Bit’s support 
for Windows operating system metrics, MacOS system metrics, and process metrics, 
practitioners can now use a single configuration schema and a single agent on all 
their client, server, and edge deployments. 

## SQL support provides more options for processing data

Fluent Bit provides granular control over processing log data with powerful plugins. 
Many of the plugins utilize regular expressions (RegEx) to parse the data, which can 
be difficult due to RegEx’s complexity. With version 3, 
[Fluent Bit adds support for SQL](https://docs.fluentbit.io/manual/stream-processing/getting-started/fluent-bit-sql), 
which can be just as powerful as RegEx but is typically easier to learn, 
especially for new developers.

Users will be able to create a new stream of data using the results from the 
SELECT statement, ensuring the integrity of the original data stream. In addition, 
the results can be aggregated with standard functions such as AVG, COUNT, MIN, 
MAX, and SUM. 

Last year, Fluent Bit added the ability to create custom processing plugins using Lua 
scripts or WebAssembly (Wasm). The addition of SQL is another step toward providing 
users with the options and flexibility they desire.

## HTTP/2 means greater interoperability

The Fluent Bit maintainers are committed to supporting open standards, and with v3 we 
have added support for HTTP/2. As a result, Fluent Bit can now receive signals emitted 
over the HTTP/2 protocol, including Prometheus Remote-Write and OTLP. 

As organizations increasingly adopt open standards, Fluent Bit’s support for open 
standards helps ensure that the full observability tech stack works together and 
that organizations can choose the best tools to fit their needs. Support for HTTP/2 
output from Fluent Bit is on the near-term roadmap. 

## Reasons behind Fluent Bit’s success

Fluent Bit is a lightweight and highly scalable processor and forwarder for logs, 
metrics, and traces. It is built for cloud and containerized environments and can 
handle massive volumes of telemetry data created in cloud-native environments. 
As cloud native applications have become the norm, the adoption of Fluent Bit 
has grown dramatically. 

Fluent Bit’s adoption can be attributed to several factors:

* **Designed for cloud native:**  Microservices are now the defacto architecture for 
delivering cloud-native applications.  With its small memory and CPU utilization 
footprint, Fluent Bit was created with those environments specifically in mind.
* **Built for Kubernetes and containers:**  Fluent Bit has proven its ability to process 
large amounts of data efficiently with a super fast, lightweight, and highly scalable 
footprint. Today, organizations use Fluent Bit to process terabytes of data per day.
* **Telemetry data is more important than ever:**  Enterprises increasingly store and 
analyze vast amounts of data as part of their observability efforts to reduce application 
downtime and incident response time.
* **Trusted by the major cloud providers:** Fluent Bit is deployed in major Kubernetes 
distributions, including Google Kubernetes Engine (GKE), AWS Elastic Kubernetes Service 
(EKS), and Azure Kubernetes Service (AKS).

## A bit about Fluent Bit’s history

Fluent Bit is a [CNCF-Graduated project](https://www.cncf.io/projects/) under the 
umbrella of Fluentd, alongside other foundational technologies such as Kubernetes 
and Prometheus.  It was initially created to be a lighter-weight version of Fluentd 
for collecting and forwarding logs from embedded or edge devices where deploying 
Fluentd would be impractical or even impossible due to limited system resources. 
It has since evolved and is now capable of collecting logs, metrics, and traces, 
processing them in mid-stream, and routing them to any number of backends.

## Learn more

To learn more about Fluent Bit, visit the [project website](https://fluentbit.io/) 
or the recently launched [Fluent Bit Academy](https://calyptia.com/on-demand-webinars) 
where you will find hours of on-demand training videos covering best practices and 
how-to’s on advanced processing, routing, and all things Fluent Bit. 
Here’s a sample of what you can find there:

* [Fluent Bit for Windows](https://calyptia.com/on-demand-webinars/on-demand-fluent-bit-for-windows)
* [Getting Started with Fluent Bit and OpenSearch](https://calyptia.com/on-demand-webinars/getting-started-with-fluent-bit-and-opensearch)
* [Getting Started with Fluent Bit and OpenTelemetry](https://calyptia.com/on-demand-webinars/getting-started-with-fluent-bit-and-opentelemetry)

We also invite you to join the vibrant Fluent community. Visit the 
[project’s GitHub repository](https://github.com/fluent/fluent-bit) to 
learn how to become a contributor. Or join [Fluent Slack](https://launchpass.com/fluent-all) 
to connect with thousands of fellow Fluent Bit and Fluentd users who help one another 
with issues and discuss the projects’ roadmaps. 

