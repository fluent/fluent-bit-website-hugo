---
title: 'OpenTelemetry with Fluent Bit'
description: >
  Fluent Bit is a vendor-agnostic Telemetry agent with full support for
  OpenTelemetry Protocol (OTLP) and overall processing capabilities.
date: 2024-11-12
headerTheme: light
#herobg: "/images/hero@2x.jpg"
herobg: "/images/hero@2x.jpg"
---

## OpenTelemetry

<br>

OpenTelemetry provides an open-source, vendor-neutral standard for Logs, Metrics, and Traces. It defines an open source schema for these telemetry data types, collectively known as signals, ensuring consistency and compatibility across diverse implementations. Because of its vendor-neutral approach, OpenTelemetry has been widely adopted and supported by numerous platforms and tools in the observability ecosystem. __Fluent Bit__ is one such implementation, offering a __lightweight__ and efficient solution for __processing__ and forwarding telemetry data in line with OpenTelemetry standards.

Each telemetry type in OpenTelemetry has a specific schema that defines the structure and format of the data. This schema ensures that telemetry data is consistent and interoperable across different systems, enabling seamless integration:

![/images/opentelemetry/otel_schema.png](/images/opentelemetry/otel_schema.png)

While schemas ensure consistency and interoperability of telemetry data, the OTLP (OpenTelemetry Protocol) handles the transport.

#### OTLP - Transport Protocol

A key component of OpenTelemetry is __OTLP__ (OpenTelemetry Protocol), a gRPC-based transport protocol designed for the structured transmission and reception of telemetry data like logs, metrics, and traces. OTLP ensures data consistency and interoperability across different systems:

![/images/opentelemetry/otel_otlp.png](/images/opentelemetry/otel_otlp.png)

<br>

#### Instrumentation Libraries

OpenTelemetry offers instrumentation libraries for a wide range of programming languages and frameworks. These libraries make it easy for developers to instrument their applications, collecting telemetry data in the OpenTelemetry format and seamlessly sending it to the desired backend using the OTLP protocol:

![/images/opentelemetry/otel_sdk.png](/images/opentelemetry/otel_sdk.png)

While instrumentation libraries are essential for collecting telemetry data within applications, a telemetry agent is needed to process and forward this data to the desired backend: Fluent Bit.

## Fluent Bit

In the OpenTelemetry ecosystem Fluent Bit is a high-performance, vendor-agnostic telemetry agent that fully supports the OpenTelemetry Protocol (OTLP). As an alternative to the OpenTelemetry Collector, Fluent Bit offers robust processing capabilities, efficient resource utilization, and seamless integration with various backends.

![/images/opentelemetry/otel_fluentbit.png](/images/opentelemetry/otel_fluentbit.png)

<br>

### Why Choose Fluent Bit for OpenTelemetry Collector ?

<br>

- **High Performance**: Designed for high throughput and low resource usage, Fluent Bit can handle large volumes of data with minimal overhead.

- **Extensive Plugin Ecosystem**: With a rich set of input, processors and output plugins, Fluent Bit can easily integrate with diverse data sources and destinations.

- **Flexible and Scalable**: Fluent Bit is lightweight and can be deployed in various environments, from edge devices to cloud infrastructures.

<br>

### Getting Started with Fluent Bit and OpenTelemetry

To get started with Fluent Bit and OpenTelemetry, follow these steps:

1. **Install Fluent Bit**: Follow the [installation guide](https://docs.fluentbit.io/manual/installation) for your platform.

2. **Configure OpenTelemetry**: Set up Fluent Bit to use the OpenTelemetry Protocol by configuring the appropriate [input (source)](https://docs.fluentbit.io/manual/pipeline/inputs/opentelemetry) and [output (destination)](https://docs.fluentbit.io/manual/pipeline/outputs/opentelemetry) plugins.

3. **Run Fluent Bit**: Start Fluent Bit and verify that it's collecting and forwarding telemetry data as expected.

For detailed configuration instructions, visit the [Fluent Bit documentation](https://docs.fluentbit.io/manual/pipeline/outputs/opentelemetry).

### Community and Support

Join our community to stay updated, share your experiences, and get support from other Fluent Bit users:

- **[Slack](https://launchpass.com/fluent-all)**: Engage with experts and other community members.
- **[Twitter](https://twitter.com/fluentbit)**: Follow us for the latest news and announcements.
- **[Newsletter](https://www.fluentd.org/newsletter)**: Sign up for updates on product information, events, and contributions.

---

