---
title: "Fluent Bit completes security audit"
date: 2025-12-09
description: "Fluent Bit recently completed a security audit by Ada Logics, addressing 11 issues ranging from low to high severity, and improving its fuzzing capabilities."
tags: ["fluentbit", "security", "audit", "OSS-Fuzz", "CNCF", "Fuzzing"]
image: "/images/blog/1701353456-general-fluent-bit-preview-card.png"
author: "David Korczynski, Adam Korczynski"
herobg: "/images/blog/1689182792-background-fluent-bit.png"
---

Fluent Bit is a lightweight, high-performance agent for collecting and forwarding logs, metrics, and traces. With a footprint of less than 5 MB, it efficiently handles millions of events per second while keeping CPU and memory usage low, making it well-suited for cloud-native and edge environments.

Supporting more than 70 input, filter, and output plugins, Fluent Bit integrates with tools such as Elasticsearch, Loki, Splunk, and Kafka. It is used at scale by organizations including AWS, Microsoft, and LinkedIn, where it reliably processes billions of records daily.

Fluent Bit recently completed a security audit carried out by Ada Logics. The audit uncovered 11 issues ranging from low to high severity, all of which were remediated with the conclusion of the audit. The audit was a coordinated collaboration among the Cloud Native Computing Foundation, the Fluent Bit maintainers, and Ada Logics, with a primary focus primarily of improving Fluent Bit’s fuzzing capabilities. Fluent Bit has been a long-time adopter of fuzzing and has been integrated into OSS-Fuzz since early 2020. Given its effectiveness a major part of the audit was dedicated to improving these efforts further.

### Key findings included:

*   **Monkey server**: Fluent Bit’s small, embedded HTTP server that exposes a flexible API and aims to behave as a full HTTP development framework. The audit found a use-after-free on the heap in this server, which caused Fluent Bit to read memory that had already been freed.
*   **cmetrics**: cmetrics is a library that Fluent Bit uses to represent, manage, and export metrics in a structured way. It draws inspiration from Prometheus’s Go Client API design and offers the internal structures for handling metrics. The audit revealed two issues in the cmetrics tool - a NULL-dereference and a segv from dereferencing uninitialized memory that provide the conditions for triggering denial of service of Fluent Bit.
*   **ctraces**: ctraces is a library that implements low-level routines for managing trace context, encoding trace spans, decoding trace payloads, and other tasks needed for processing trace telemetry. The audit uncovered two NULL-dereferences that could cause denial of service of a running Fluent Bit instance.
*   **Config reading and processing**: Several issues were identified in configuration processing that could lead to denial-of-service conditions or of out-of-bounds memory reads. While these are less severe since untrusted users typically cannot upload configurations, addressing them is important to weed them out to ensure the project’s long-term security and stability.

All issues identified in the audit have been patched. The new fuzzers from the audit are now a part of Fluent Bit’s fuzzing suite that is integrated into OSS-Fuzz. The new fuzzers continue to run with OSS-Fuzz’s extensive compute power and will continue to test Fluent Bit.

You can read the full security audit report here: <https://github.com/fluent/fluent-bit/blob/2f7791edbcf7c942789c41ba7abe7e1a8f9d7ccf/doc-reports/cncf-security-audit.pdf>.
