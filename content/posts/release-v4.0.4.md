---
title: "Fluent Bit v4.0.4"
date: 2025-07-09
description: "Fluent Bit v4.0.4 introduces powerful OTLP improvements, Lua metadata access, encoding support for Asian markets, and secure MSK IAM authentication."
tags: ["fluentbit", "release", "opentelemetry", "kafka", "lua", "aws", "encoding"]
image: "../images/blog/1701353456-general-fluent-bit-preview-card.png"
author: "Eduardo Silva Pereira"
herobg: "/images/blog/1689182792-background-fluent-bit.png"
---

# 🔥 Fluent Bit v4.0.4 is here !

> **TL;DR:** Fluent Bit v4.0.4 introduces a powerful **OpenTelemetry interface** for better encoding and decoding, enhanced **Lua scripting with OTLP metadata access**, **character encoding conversion** for non-UTF-8 logs, and **AWS IAM support for Kafka/MSK** — plus performance boosts, NFS-tail fixes, and dozens of other improvements.

---

### New OpenTelemetry Interface

Fluent Bit continues to strengthen its position as a modern **OpenTelemetry data pipeline**. In v4.0.4, we introduce a new **internal OpenTelemetry interface** for handling encoding and decoding — making OTLP log processing faster, safer, and more testable.

#### Key Improvements:


- 🔄 **Unified OTLP handling** across JSON and Protobuf
- ✅ **Cleaner separation of concerns** between input, processing, and output stages
- 🧪 Better unit testing and error reporting for encoding issues
- 🔧 New support for custom `logs_body_key` in `out_opentelemetry` and `in_opentelemetry` to structure OTLP log bodies as needed

This foundation enables stronger support for trace/metrics/logs ingestion, and positions Fluent Bit as a robust forwarder in OTLP-native environments.

---

#### 🧠 Lua Filter Superpowers: OTLP Metadata + Group Access

In v4.0.4, the Lua filter is now OpenTelemetry-aware, enabling advanced **record-level and group-level processing**. This is ideal for modifying OTLP logs on the fly based on resource or scope attributes.

#### New Function Signature:

```lua
function cb(tag, timestamp, group, metadata, record)
```
<br />

##### What You Can Do:

- Inject `resource.attributes` (like service name) into log records
- Modify OTLP severity levels or labels
- Return multiple records and metadata per invocation
- Mix both simple and complex scripting with backward compatibility

##### Example: Adjust OTLP Severity and Add Service Name
```lua
function cb(tag, ts, group, metadata, record)
  if group['resource']['attributes']['service.name'] then
    record['service_name'] = group['resource']['attributes']['service.name']
  end

  if metadata['otlp']['severity_number'] == 9 then
    metadata['otlp']['severity_number'] = 13
    metadata['otlp']['severity_text'] = 'WARN'
  end

  return 1, ts, metadata, record
end
```

> 🧩 Compatible with OpenTelemetry Collector logs and any OTLP log producers.

---

#### 🌏 Encoding Support for Asian Markets: Native Transcoding

For many organizations in Asia, logs are still written using encodings like **GBK**, **Big5**, or **Shift_JIS** — leading to `文字化け` (mojibake) when sent to UTF-8-based backends.

Fluent Bit now includes a **native character encoding engine** in `in_tail`. With the new `generic.encoding` property, you can **transcode logs to UTF-8** as they are read.

##### Example: Transcode from GBK to UTF-8

```yaml
pipeline:
  inputs:
    - name: tail
      path: /var/log/my_legacy_app.log
      parser: json
      generic.encoding: GBK

  outputs:
    - name: stdout
```

> 🔁 Ensures clean UTF-8 logs for Elasticsearch, OpenSearch, S3, or any backend.
> 🎌 Perfect for smooth Fluentd → Fluent Bit migrations.

---

#### 🔐 Kafka + AWS MSK IAM + Performance Boosts

Fluent Bit now includes **native support for AWS IAM-based authentication** when talking to **Amazon MSK** (both input and output plugins).

##### Highlights:
- 🛡️ Uses AWS SigV4 signing via **OAuthBearer**
- ☁️ Works with **EC2 IAM roles, STS tokens, and serverless MSK**
- 🧼 Stateless token refresh implementation using `librdkafka`

No more credentials in configs — just IAM and go.

##### Bonus: `enable_auto_commit` for Kafka Input

Kafka consumers now support:
```yaml
enable_auto_commit: true
```

- `false` (default) = safety-first, only commit after full processing
- `true` = **performance-oriented**, batch commit behavior, better throughput

Use this if your backend can tolerate replays and you want maximum ingestion performance.

---

#### 🧱 in_tail File Rotation Improvements (NFS-Friendly)

Working with **log files on NFS or remote mounts** can be tricky — especially when files are rotated or overwritten.

In v4.0.4, `in_tail` now uses **`fstat()`-based rotation detection**, making it much more reliable in environments like:
- 📁 Network File Systems (NFS)
- 🪵 Centralized log directories
- 🐳 Containerized applications with volume mounts

> 🧯 Prevents duplicate logs, dropped records, or missed tailing when files are moved or rotated externally.

---

#### ⚙️ Other Highlights

##### Plugins
- **out_opentelemetry**: fixes for retry, metadata merging, and grouping
- **out_loki**: prevents race conditions when multiple workers use `Remove_Keys`
- **filter_modify / filter_lua**: memory fixes and cleanup
- **in_kafka / out_kafka**: improved AWS MSK integration and token validation

##### Core & Build System
- 📦 `librdkafka` upgraded to **v2.10.1**
- 🧪 New tests for encodings, filters, upstream behavior
- 🧵 Better upstream connection reuse logic
- 🐧 Support for **Rocky Linux** and **AlmaLinux** in installer
- 🧹 Code cleanup and memory leak fixes across the board

---

### 🛠️ How to Upgrade

Fluent Bit v4.0.4 is available now:
- 📦 [Download Packages](https://fluentbit.io/download/)
- 🐳 [Docker Images](https://hub.docker.com/r/fluent/fluent-bit)
- 📚 [Documentation](https://docs.fluentbit.io/)

---

### 🙌 Thanks to the Community

This release wouldn’t be possible without contributions, bug reports, and feedback from our users and partners around the world.

