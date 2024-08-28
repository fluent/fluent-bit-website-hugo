---
title: "Meet Fluent Bit v2.2!"
date: "2023-11-16"
description: "Announcing the release of Fluent Bit v2.2! Watch the project update from Observability Day at KubeCon and read about some of the highlights from the new release."
image: "/images/blog/1700159205-fluent-bit-2-2.png"
author: "Erik Bledsoe"
canonicalUrl: "https://chronosphere.io/learn/fluent-bit-v2-2/"
herobg: "/images/blog/1689182792-background-fluent-bit.png"
---
*This post is [republished from the Chronosphere blog](https://chronosphere.io/learn/fluent-bit-v2-2/). With [Chronosphere’s acquisition of Calyptia](https://chronosphere.io/news/chronosphere-acquires-calyptia/) in 2024, Chronosphere became the [primary corporate sponsor of Fluent Bit](https://chronosphere.io/fluent-bit/). Eduardo Silva — the original creator of Fluent Bit and co-founder of Calyptia — leads a team of Chronosphere engineers dedicated full-time to the project, ensuring its continuous development and improvement.*

Last week Calyptia announced the release of Fluent Bit v2.2. at KubeCon + CloudNativeCon North America. Eduardo Silva, original creator of Fluent Bit and Calyptia CEO, presented highlights from the new release at Observability Day, a KubeCon co-located event. You can watch the [full video here](https://chronosphere.io/learn/fluent-bit-v2-2/). 

Here are some highlights of the new release.

## New sysinfo filter

System information, including the hostname, the operating system, the kernel version, and the version of Fluent Bit running, is often invaluable when trying to diagnose an issue. Too often, though, system information is unavailable to downstream search and analytics platforms. The new [Sysinfo plugin](https://docs.fluentbit.io/manual/pipeline/filters/sysinfo) can automatically gather and enrich your logs with system information.

![chart showing how data is enriched with system information](/images/blog/1700159200-fluent-bit-2-2-sysinfo.png)

## Support for cgroups v2 in Docker

Fluent Bit already supported cgroups v2 for Podman. With this new release, Fluent Bit also adds support cgroups v2 for Docker containers. Now Fluent Bit offers parity for cgroups support across Podman and Docker with support for cgroups v1 and v2 on both platforms.

![diagram](/images/blog/1700159194-fluent-bit-2-2-cgroups.png)

## New YAML configuration format

Classic Fluent Bit configuration files use a “YAML-inspired” format. Now you can [use true YAML format](https://docs.fluentbit.io/manual/administration/configuring-fluent-bit/yaml/configuration-file). The classic format files will still work, and you can even use a mixture of the two formats for different config files for the same Fluent Bit instance. The change represents the project’s dedication to open standards.

## Collect more metrics from Windows and (soon) macOS

The Node Exporter Metrics plugin is now able to collect extended metrics from Windows, giving you more insight into what is happening with your Windows-based machines. Support for macOS will be rolling out shortly. 

## New native integration for Loki

We’ve completely reworked our integration with Grafana Loki to make it native. The original Loki Golang plugin is being deprecated. 

## New Kubernetes Events plugin

The new Kubernetes Events plugin lets you collect cluster events through the API server. You can collect events from all namespaces, or limit it to specific namespaces. 

![Diagram](/images/blog/1700159188-fluent-bit-2-2-kubernetes-events.png)

## Significant performance improvements

Fluent Bit was already [highly performant](https://calyptia.com/blog/benchmarking-fluent-bit). That said, demands on data pipelines are growing and the Fluent Bit technology is advancing to keep up with these increased demands. We've improved Fluent Bit's performance under the most demanding use cases. Testing of Fluent Bit v2.2 using multiple filters with hybrid mem+fs storage and checksums ON showed up to a 6X performance improvement based upon CPU and memory usage. 

## Try out Fluent Bit v2.2 for yourself

We’ve covered just the highlights of what Fluent Bit v2.2 offers. [Check out the full changelog](https://github.com/fluent/fluent-bit/releases/tag/v2.2.0) for all the nitty gritty details. 

To see for yourself how Fluent Bit can enhance your data pipelines, [try it now](https://docs.fluentbit.io/manual/installation/getting-started-with-fluent-bit). 

![Half-day Training](/images/blog/1697140572-summerseries_website-image.png)

New to Fluent Bit? Check out our [Fluent Bit half-day training](https://calyptia.com/events/fluent-bit-half-day-training), available on demand. The three-part series covers:

1. Fluent Bit 101 - The basics of data collection and routing & common architectures
2. Fluent Bit Processing - Processing streaming data (including redaction, enrichment, and dropping noisey data)
3. Fluent Bit Operations & Monitoring - Metrics, patterns, & agents

## Did you know?

[Fluent Bit](https://fluentbit.io/) has over [10 billion Docker pulls](https://chronosphere.io/learn/fluent-bit-surpasses-10-billion-docker-pulls/) and is the industry standard open-source solution for

processing and forwarding logs and metrics. It is deployed in major Kubernetes distributions, including Google Kubernetes Engine (GKE), AWS Elastic Kubernetes Service (EKS), Azure Kubernetes Service (AKS), and more.

