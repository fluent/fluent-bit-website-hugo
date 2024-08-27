---
title: "Fluent Bit surpasses 10 billion Docker pulls!"
date: "2023-09-25"
description: "That’s billion, with a “B.” It is currently averaging over 23 million pulls per day. That’s 270 pulls every second, 24/7"
image: "/images/blog/1695998714-10-billion-party.png"
author: "Erik Bledsoe"
canonicalUrl: "https://calyptia.com/blog/fluent-bit-surpasses-10-billion-docker-pulls"
---
*This post was originally published on the Calyptia blog. With [Chronosphere’s acquisition of Calyptia](https://chronosphere.io/news/chronosphere-acquires-calyptia/) in 2024, Chronosphere became the [primary corporate sponsor of Fluent Bit](https://chronosphere.io/fluent-bit/)</a>. Eduardo Silva — the original creator of Fluent Bit and co-founder of Calyptia — leads a team of Chronosphere engineers dedicated full-time to the project, ensuring its continuous development and improvement.*

Fluent Bit recently surpassed 10 billion pulls from Docker Hub. That’s *billion*, with a “B.” 🎉

As the creators and maintainers of [Fluent Bit](https://fluentbit.io/), we at Calyptia are stunned and humbled by its success.

![illustration of a party with a 10 billion banner](/images/blog/1695998714-10-billion-party.png)Originally released in the Docker registry in 2017, Fluent Bit surpassed one billion Docker pulls in March 2022. Since then, the pulls have accelerated rapidly, adding an additional nine billion in eighteen months. 

To help put those numbers in perspective:

* Fluent Bit has added over five billion pulls in 2023 so far
* It is currently averaging over 23 million pulls per day
* That’s 270 pulls every second, 24/7

## Wait, how is that possible?

First, you can see the exact count using the [Docker API](https://hub.docker.com/v2/repositories/fluent/fluent-bit/)—look for the pull\_count key. Next, 10 billion Docker pulls does not mean that 10 billion developers have downloaded Fluent Bit from Docker. Most of those pulls will be automated as organizations scale up their infrastructure to meet demand. It’s easy for large companies with complex infrastructures to rack up thousands of pulls in a day under some conditions.  

Amazingly, the numbers we can track using the Docker API don’t tell the entire story. They do not, for example, include private repositories where organizations clone and store the official image, which is how most large enterprises enforce version control. Also not counted in the above are the instances when Fluent Bit is deployed “behind the scenes” as part of a cloud provider’s service, as it is by AWS, Google, and Azure. So the real numbers are undoubtedly higher, probably significantly higher. But what’s a few billion pulls, give or take?

## Why is Fluent Bit so Popular?

* **Designed for Cloud-Native:** Microservices are now the defacto architecture for delivering cloud-native applications.  With its small memory and CPU utilization footprint, Fluent Bit was created with those environments specifically in mind.
* **Built for Kubernetes and Containers:**  Despite its low resource utilization, Fluent Bit has proven its ability to [process large amounts of data efficiently](https://calyptia.com/blog/benchmarking-fluent-bit). Because of its performance at scale, Fluent Bit is embedded in major Kubernetes distributions, including those from AWS, GCP, and Azure.
* **Observability and Security Data is more important than ever:**  Enterprises are increasingly storing and analyzing event data as part of their observability and security efforts to reduce application downtime and incident response time.

## A bit about Fluent Bit’s history

Fluent Bit is a [CNCF-Graduated project](https://www.cncf.io/projects/) under the umbrella of Fluentd, alongside other foundational technologies such as Kubernetes and Prometheus.  It was originally created to be a lighter-weight version of Fluentd for collecting and forwarding logs from Internet of Things (IoT) devices and containers where deploying Fluentd would be impractical or even impossible due to limited system resources. It has since evolved and is now capable of collecting logs, metrics, and traces, processing them in mid-stream, and routing them to any number of backends.

## How to manage all those Fluent Bit instances?

With so many Fluent Bit agents deployed across increasingly complex architectures, enterprises may struggle with managing them all. To address this need, Calyptia, the creators of Fluent Bit, has introduced Fluent Bit fleet management into [Calyptia Core,](https://calyptia.com/products/calyptia-core) our commercial product that simplifies data management for your observability and SIEM pipelines. Fleet management enables remote management for edge pipelines. Teams can configure and manage thousands of instances of Fluent Bit across the enterprise from one central location, significantly increasing efficiency and reducing the risk of misconfiguration. To learn more, [book a demo](https://calyptia.com/demo). 

