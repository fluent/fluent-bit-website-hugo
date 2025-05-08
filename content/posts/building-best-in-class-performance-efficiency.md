---
title: "Fluent Bit v3.2: Building on best-in-class performance and efficiency"
date: "2025-03-06"
description: "This week the Fluent Bit maintainers announced the launch of Fluent Bit v3.2."

image: "../images/blog/1701353456-general-fluent-bit-preview-card.png"
author: "Carolyn King | Head of Community & Developer | Chronosphere"
herobg: "/images/blog/1689182792-background-fluent-bit.png"
---

### New release: Fluent Bit v3.2

This week Fluent Bit maintainers are excited to announce the launch of Fluent Bit v3.2. This release delivers major performance improvements, increased efficiency, new signal support, and new capabilities for OpenTelemetry, YAML and eBPF. With v3.2, the Fluent Bit project continues to innovate and deliver the new capabilities and ecosystem integrations required to meet the complex needs of observability and security teams.

Built on the best practices and learnings from Fluentd, Fluent Bit was created to be a light-weight version able to collect and forward logs from Internet of Things (IoT) devices and containers, where deploying Fluentd would be impractical due to limited system resources.

Since its inception, Fluent Bit has expanded its capabilities to include collecting logs, metrics, and traces, providing in-stream processing and multi-routing capabilities, and much more.

After achieving CNCF graduated project status in 2019, Fluent Bit hit 1 billion downloads in 2022 and adoption has since skyrocketed from 1 to over 15 billion downloads today. This growth has been largely fueled by the adoption of Kubernetes and Fluent Bit’s compatibility with cloud native environments.

### Key drivers of Fluent Bit’s global adoption include:

1. Designed for Cloud-Native:Microservices are now the de facto architecture for delivering cloud-native applications. With its small memory and CPU utilization, Fluent Bit is optimized for those environments.
2. Built for Kubernetes and Containers: Fluent Bit is able to process large amounts of data efficiently. Because of its scalability, Fluent Bit is embedded in major Kubernetes distributions, including those from AWS, GCP, and Azure.
3. Observability and security data is more important than ever: Enterprises are increasingly storing and analyzing event data as part of their observability and security efforts to reduce application downtime and incident response time.

### Let’s take a look at the Fluent Bit v3.2 TLDR:

1. Performance improvements with SIMD and Multi-threading
2. New signal type support for Blob
3. Fluent Bit + eBPF
4. Interoperable ecosystem with new OTel and YAML Capabilities

#### 1. Performance improvements with SIMD and Multi-threading

While Fluent Bit throughput and resource usage is already best in class, v3.2 introduces internal enhancements to optimize processing of log files as well as new defaults that ensure the best performance out of the box.

![forwarder-aggregator](/images/blog/1FluentBitv3.2-1024x538.png)

With v3.2, Fluent Bit now comes standard with Single Instruction Multi Data (SIMD) support for log processing and parsing without any additional work from users, providing immediate performance benefits with this release.

In addition, 3.2 builds on Fluent Bit’s core with continued default multi-threading for inputs, outputs, and processing of observability signal types, including logs, metrics, and traces.

#### 2. New signal type support for Blob

Fluent Bit 3.2 includes two new signal types with blob and eBPF. While Fluent Bit has traditionally been used to move petabytes of machine and observability data, new requirements from users include binary data like photos, videos, and files. A common use case – AI companies require the ability to move large volumes of pictures and videos in order to train their AI models.

![forwarder-aggregator](/images/blog/2FluentBitv3.2-1024x571.png)

With blob signal support Fluent Bit can be used for moving massive files to storage destinations such as Azure Blob. This is particularly relevant for applications that need to collect and process photos and video files. In addition to AI, this includes autonomous vehicles, transportation, healthcare, retail, industrial automation, and more. Fluent Bit offers a turnkey way for companies to collect this data and the ability to send it to multiple backends.

#### 3. Fluent Bit + eBPF

Extended Berkeley Packet Filter (eBPF) is bringing new capabilities in security and advanced observability to Fluent Bit. In v3.2, Fluent Bit adds the ability to run eBPF programs to ingest data into the pipeline.

### How fluent-bit can benefit from eBPF

* Use the existing tracepoints and tracing hooks in the kernel to extend the number of input events generators
* in_docker_events
* in_kubernetes_events
* […]
* Trace events from all system calls
* Trace events from other userland programs through uprobes and ftrace tracing points
* Trace and gain visibility into xdp/sockets information
* Trace and gain visibility into performance counters
* Allow users to load custom eBPF programs for custom needs
* Be independent of external tracing tools

This includes out-of-the-box eBPF capabilities for users to plug in their own eBPF programs. New eBPF capabilities also provide integrations that enable developers to plug in to other CNCF eBPF projects, such as Falco and Tracee, for security use cases.

#### 4. An interoperable ecosystem with new OTel and YAML capabilities

Since its inception, Fluent Bit was designed for extensibility and to be interoperable with the best technologies in the space.

Eduardo Silva, original creator of Fluent Bit, shared “From the beginning, Fluent Bit was built to integrate with best in class technologies and open source standards, enabling users to build the tech stack that is best for them.”

With the rise of the OpenTelemetry protocol as a standard for observability,  Fluent Bit continues its integration and standardization with OTel. This includes increased compatibility across logs, metrics, and traces.

The interoperability of Fluent Bit and OTel means that teams can choose the best technology for their needs. For example, let’s say a developer needs a lighter and more performant agent than the OTel collector but they need to be able to convert the data to an OTel format.

The user can now use the Fluent Bit agent to collect the data, and then leverage the OTel Envelope processor to convert the logs to the correct format for an OTel backend.

![forwarder-aggregator](/images/blog/4FluentBitv3.2-1024x391.png)

The interoperability of Fluent Bit and OTel is particularly important for users who are (1) migrating to Open Source standards and (2) implementing the OpenTelemetry protocol. Fluent Bit v3.2 introduces the following OTel capabilities:

1. The OpenTelemetry Envelope processor. The envelope processor enhances the handling of non-native OpenTelemetry data. The new OpenTelemetry envelope processor allows any of the Fluent bit log sources to automatically convert into an OTel schema without any additional work from the user side. This enables users to send non-OpenTelemetry logs to an OpenTelemetry backend.

2. Content Modifier for OpenTelemetry Logs. With additions to the Content Modifier users can now recognize additional OTel data, including resources and scopes. Users now have the ability to manage OpenTelemetry logs, resources and scopes and to enrich or transform logs that are under an OTel log schema.

New additions to YAML also make it easier for developers to adopt and leverage Fluent Bit. As many developers know, YAML has been the standard for Kubernetes configuration and many other applications for years.

![forwarder-aggregator](/images/blog/5FluentBitv3.2-1024x801.png)

Previous Fluent Bit versions included some YAML compatibility, but Fluent Bit v3.2 now includes full support for YAML in every part of the Fluent Bit pipeline, including parsers, configuration, processors, and settings.

This allows a single unified configuration language across both Fluent Bit and Kubernetes resources which means that developers don’t need to recreate efforts inside of Fluent Bit.

### Conclusion

Fluent Bit v3.2 pushes the boundaries of performance, versatility, and interoperability in cloud-native observability. With significant enhancements like SIMD support, eBPF integrations, and full OpenTelemetry compatibility, this release ensures Fluent Bit continues to meet the evolving needs of today’s observability and security teams. Whether you’re moving large datasets or optimizing Kubernetes workflows, Fluent Bit v3.2 is equipped to help you achieve more with less.
