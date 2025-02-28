---
title: "Meet Fluent Bit v3.2.7!"
date: "2025-02-28"
description: "Announcing the release of Fluent Bit v3.2.7 bringing new features, critical security patches, and performance improvements."

image: "../images/blog/1701353456-general-fluent-bit-preview-card.png"
author: "The Fluent Bit Team"
herobg: "/images/blog/1689182792-background-fluent-bit.png"
---

# Fluent Bit 3.2.7 Release: Enhancements and Security Fixes

Fluent Bit, a fast and lightweight data processor and forwarder for Linux, BSD, Windows and macOS, has [released version 3.2.7](https://fluentbit.io/announcements/v3.2.7/). This new release brings several important updates and enhancements, particularly focused on security patches, performance improvements, and feature enhancements. Let's dive into the key highlights of this release.

### **Security Fixes**

Security is a top priority for any software, and Fluent Bit has been proactive in addressing potential vulnerabilities. The version 3.2.7 release resolves two critical security issues associated with the OpenTelemetry and Prometheus Remote Write input plugins.

1. **OpenTelemetry Input Plugin Security Fix (**[CVE-2024-50609](https://nvd.nist.gov/vuln/detail/CVE-2024-50609)**)**  
   A vulnerability was identified within the OpenTelemetry plugin. While it did not directly compromise the data collection process, attackers could exploit this vulnerability under specific conditions. With this update, the plugin has been reinforced against potential threats, ensuring better protection of telemetry data.  
2. **Prometheus Remote Write Input Plugin Security Patch (**[CVE-2024-50608](https://nvd.nist.gov/vuln/detail/CVE-2024-50608)**)**  
   Another important fix is for the Prometheus Remote Write input plugin. This vulnerability could have impacted users working with Prometheus data integrations. The update addresses this issue, making sure that remote writes to Prometheus are secure and uninterrupted. Users who rely on Prometheus for monitoring and alerting can now rest assured that their data pipelines are safe.

### **Improvements and Core Updates**

Alongside the security fixes, Fluent Bit 3.2.7 introduces several performance improvements that help optimize memory management and enhance the overall user experience:

1. **Deadlock Prevention with New Management Signal**  
   One of the key improvements in this release is the introduction of a new log management signal aimed at preventing deadlocks. It should be noted that this only affected special configurations at start time.  
2. **Enhanced zstd Compression**  
   Fluent Bit continues to optimize its handling of compressed data formats. With the addition of enhanced [zstd](http://www.zstd.net/) compression support, users can now expect better data compression ratios, reducing storage requirements and improving transmission speed when dealing with large volumes of log or trace data.  
3. **Improved Memory Allocation in Plugins**  
   Efficient memory usage is a hallmark of Fluent Bit's design, and the 3.2.7 update brings even better memory allocation handling in various input and output plugins. This improvement ensures that Fluent Bit operates more efficiently under high loads, handling more data with fewer resources, and reducing the chances of out-of-memory issues in large-scale deployments.

### **Plugin Updates and New Features**

Fluent Bit's wide range of plugins received several key updates in this release, adding new functionality and improving compatibility with various systems:

1. **Kubernetes Events Plugin Fixes**  
   For users who leverage Fluent Bit in Kubernetes environments, this release provides important fixes to the Kubernetes plugin in sqlite database cleanup on restarting.   
2. **Elasticsearch and Splunk Plugin Enhancements**  
   The Elasticsearch and Splunk plugins have also been updated to support additional features and provide better compatibility with their respective systems. These improvements allow for smoother integration and more reliable data flows, making it easier for users to send their telemetry data to these monitoring platforms.  
3. **HTTP Plugin Updates**  
   Fluent Bit’s HTTP plugin has also received some attention. The plugin now boasts support for gzip and zstd compression handling, plus memory allocation fixes for better integration with web services and APIs, streamlining the process of pushing collected data to remote HTTP endpoints.  
4. **Support for OpenTelemetry JSON Traces**  
   OpenTelemetry is a popular framework for observability, and Fluent Bit has added support for OpenTelemetry JSON Traces. This feature allows users to collect trace data in the JSON format, which is widely used across the observability ecosystem. This improvement opens up new possibilities for users who are looking to enhance their tracing capabilities.  
5. **gRPC Compressed Messages Support**  
   In addition to the OpenTelemetry JSON Traces, Fluent Bit now supports compressed messages over gRPC using gzip, zstd, and snappy. gRPC is widely used for high-performance communication in microservices architectures, and this update enables users to send compressed logs and metrics over gRPC, optimizing both bandwidth and storage requirements.

### **Conclusion**

Fluent Bit 3.2.7 is a release that brings new features, critical security patches, and performance improvements. By addressing vulnerabilities in the OpenTelemetry and Prometheus plugins, enhancing memory management and compression capabilities, and introducing important updates for Kubernetes, Elasticsearch, and Splunk integrations, Fluent Bit solidifies its position as a go-to solution for efficient telemetry data collection.

For developers and organizations who rely on Fluent Bit to manage their observability data, upgrading to version 3.2.7 is highly recommended. The improvements in security, performance, and functionality ensure that Fluent Bit remains a top-tier tool for processing and transporting all your telemetry data in complex, high-performance environments. As always, this release continues to push the boundaries of what is possible with lightweight, resource-efficient observability solutions.

