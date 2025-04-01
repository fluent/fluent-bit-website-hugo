---
title: "Fluent Bit v4.0"
date: "2025-04-01"
description: "Celebrating new features and 10th anniversary"
image: "/images/blog/1691520029-banner.png"
author: "Paige Cruz"
canonicalUrl: "https://chronosphere.io/learn/fluent-bit-v4-0/"
herobg: "/images/background-fluent-bit.png"

---
# Fluent Bit v4 is here! 

Discover what's new in this release and celebrate 10 years of innovation 

*This post is [republished from the Chronosphere blog](https://chronosphere.io/learn/fluent-bit-v4-0/). 

The Fluent Bit maintainers have exciting news to share! Fluent Bit version 4 is out 
and just in time to celebrate the project's 10-year anniversary.

## The journey: From embedded logging to multi-Signal observability

With over 15 billion downloads, integration with every major cloud provider, 
and adoption by thousands of enterprises, Fluent Bit has become the production-grade 
telemetry pipeline for cloud native environments. 

Fluent Bit began in 2014 with a clear mission: create a lightweight, high-performance 
log processor that could run efficiently in resource-constrained environments. 
What distinguished Fluent Bit from the start was its commitment to performance 
and minimal resource consumption—principles that continue to guide development today.

## The project's journey has been marked by consistent growth and evolution:

![Fluent Bit timeline](/images/blog/blog-Fluent-bit-v4-timeline.png)

Throughout this journey, [Fluent Bit](https://chronosphere.io/learn/fluent-bit-v3-2/) has maintained its core values: 
high performance, lightweight footprint, and vendor neutrality. These principles have made 
it the preferred choice for organizations seeking a reliable observability foundation 
without vendor lock-in or performance compromises

## Introducing Fluent Bit v4: The next generation

Building on a decade of engineering excellence, v4 introduces transformative capabilities 
that expand what's possible with a telemetry agent while maintaining backward compatibility 
and the performance users expect. Here are four fantastic features you’ll find in v4:

1. Conditional logic for processors
2. Head and tail sampling for traces
3. Zig language support for extending Fluent Bit
4. Authentication and TLS improvements

## Adding conditional logic for processors

When  v2.12 was released, [Fluent Bit](https://chronosphere.io/fluent-bit/) introduced the concept of [Processors](https://docs.fluentbit.io/manual/pipeline/processors), 
which can enrich or transform telemetry data. Then with v3, Fluent Bit expanded 
on this foundation by introducing three key processors: Content Modifier, Metric Selector, 
and which follows last year's new support for the OpenTelemetry Envelope Enveloper. 

The ability to transform data in processors by writing rules and configurations is powerful 
but also complex - until now. With v4, Fluent Bit offers intuitive conditional logic that makes 
it dramatically easier to implement sophisticated data transformations while improving pipeline 
performance and efficiency.

Conditional logic in processors significantly reduces bottlenecks in high-volume data streams, 
which is especially powerful for use cases with heavy regex processing or complex data transformations., 

Here's an example of how you can use conditional processing to delete sensitive information 
based on user permissions:

```yaml
service:
  flush: 0.2
pipeline:
  inputs:
    - name: dummy
      dummy: '{"user": "admin", "roles": ["read", "write", "delete"], "secret": "abc123"}'
  outputs:
    - name: stdout
  processors:
    logs:
      - name: calyptia
        actions:
          - type: delete
            opts:
              key: secret
            condition:
              operator: AND
              rules:
                - field: '$roles'
                  operator: not_in
                  value: ["admin", "superuser"]
                - field: '$user'
                  operator: neq
                  value: "system"
```


##### This configuration will transform:


Example 1: 

Condition Met

```yaml
{
  "user": "admin", 
  "roles": ["read", "write", "delete"],     
  "secret": "abc123"
}
```

Secret Key Deleted

```yaml
{
  "user": "admin", 
  "roles": ["read", "write", "delete"]
}
```

Example 2: 

Condition Not Met

```yaml
{
  "user": "system", 
  "roles": ["superuser"],     
  "secret": "abc123"
}
```

No Change

```yaml
{
  "user": "system", 
  "roles": ["superuser"],     
  "secret": "abc123"
}
```


## Head and Tail sampling for traces

Fluent Bit v4 now offers a new trace sampling processor designed with a pluggable architecture, 
which allows easy extension to support multiple sampling strategies and backends. 

The two main approaches for sampling traces are head and tail sampling: 
- Head sampling makes the decision whether or not to keep a trace at the very beginning.
This is when a root span is created but before the request is actually fulfilled. 

- Tail sampling evaluates the entire trace when making a sampling decision and can inspect 
the metadata and status of traces to inform the decision. 

Here’s an example showing head sampling configured at 40%, which would result in approximately 
40 traces sampled out of an original dataset of 100. 

```yaml
pipeline:
  inputs:
    - name: opentelemetry
      port: 4318

  processors:
    traces:
      - name: sampling
        type: probabilistic
        sampling_settings:
          sampling_percentage: 40

  outputs:
    - name: stdout
      match: '*'

```


Trace data volumes can quickly become overwhelming and expensive to store. Intelligent 
sampling ensures you capture the most valuable traces for debugging and performance analysis 
while managing costs. I am excited to see expanded support for traces — it means [Fluent Bit](https://chronosphere.io/fluent-bit-academy/) continues to be a powerful choice for handling 
all your cloud native telemetry. This is an area the Fluent Bit maintainers are actively 
working on and welcome your feedback, friction logs, and feature wishlist when it comes 
to managing traces with Fluent Bit.

## Zig language support for extending Fluent Bit and developing plugins

If you’ve been wanting to contribute to or customize Fluent Bit but weren’t keen on learning 
the ins and outs of C, you’ll be excited to learn about our support for the Zig language! 
Zig is a modern systems programming language that combines C-like performance with memory 
safety and expressive syntax. This complements existing C and Lua plugin options, 
enabling developers to create high-performance extensions with fewer bugs and security vulnerabilities.

Going forward Zig will complement existing extension options like [Golang](https://docs.fluentbit.io/manual/development/golang-output-plugins), [Lua](https://docs.fluentbit.io/manual/pipeline/filters/lua) and [WASM](https://docs.fluentbit.io/manual/pipeline/filters/wasm) opening 
the door to more contributions from a broader community of developers. 

Learn more about [Zig](https://ziglang.org/) here and check out their list of [learning resources](https://ziglang.org/learn/) to get started today!

## Authentication and TLS improvements

As observability data increasingly contains sensitive information and spans multiple security domains, 
robust authentication and encryption are essential. These v4 improvements enable secure 
telemetry collection across complex environments.

First is the ability to load Bearer tokens directly from the filesystem—a big improvement 
over manual configuration. Second, the Fluent Bit maintainers have responded to community 
requests for more choice and flexibility for the cipher used to encrypt communication via TLS.


## Looking ahead: The next decade of Fluent Bit

As we celebrate this milestone, it’s amazing to watch how the community has helped 
make Fluent Bit's success possible. From individual contributors and maintainers to enterprise adopters, 
your feedback, contributions, and support have shaped this project into what it is today.

Fluent Bit v4 isn't just a culmination of our first decade—it's the foundation for the next ten 
years of innovation. We envision Fluent Bit continuing to evolve as observability practices mature, 
becoming increasingly intelligent about the data it processes while remaining true to its core principles 
of efficiency and vendor neutrality.

I recommend  upgrading to v4 today so you can experience first-hand how these transformative capabilities 
can enhance your observability practice.

## Ready to get started? 
Learn more about Fluent bit v4, join our growing community, and visit our documentation 
to learn more about upgrading. Here's to the next decade of observability excellence with Fluent Bit!

- [Learn more in our next webinar about Fluent Bit v4](https://go.chronosphere.io/fluent-bit-10-year-journey-the-road-ahead-with-v4.html)

- [Join our growing community on Slack!](https://www.launchpass.com/fluent-all)

- [Upgrade to Fluent Bit v4](https://docs.fluentbit.io/manual/v4)

