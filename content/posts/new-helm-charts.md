---
title: "New Fluent Bit Helm Charts"
date: 2026-03-23
description: "Fluent Bit has new Helm charts architected for specific use-cases; the aggregator and collector Helm charts are now generally available on Artifact Hub."
tags: ["fluentbit", "helm", "chart", "aggregator", "collector", "artifacthub"]
image: "/images/blog/1701353456-general-fluent-bit-preview-card.png"
author: "Steve Hipwell, Patrick Stephens"
herobg: "/images/blog/1689182792-background-fluent-bit.png"
---

The Fluent Bit team is excited to announce the general availability of two new Helm charts, designed to cater to specific use-cases in log aggregation and collection. The [Fluent Bit Aggregator Helm Chart](https://artifacthub.io/packages/helm/fluent-bit-aggregator/fluent-bit-aggregator) and the [Fluent Bit Collector Helm Chart](https://artifacthub.io/packages/helm/fluent-bit-collector/fluent-bit-collector) are now available on [Artifact Hub](https://artifacthub.io/), providing users with streamlined deployment options for their logging infrastructure.

These charts originated in the [stevehipwell/helm-charts](https://github.com/stevehipwell/helm-charts) repository, where they have been developed and used for the past 5 years. They have now been adopted into the official [fluent/helm-charts](https://github.com/fluent/helm-charts) repository, making them part of the core Fluent Bit project. The existing [fluent-bit](https://artifacthub.io/packages/helm/fluent/fluent-bit) Helm chart remains available, but users are encouraged to migrate to the new charts for a better experience.

We encourage users to explore these new charts and take advantage of the enhanced capabilities they offer for log management in Kubernetes environments.

## Why New Charts?

The original Fluent Bit Helm chart is general-purpose, but running Fluent Bit as a log collector on each node is a fundamentally different use-case from running it as a centralized log aggregator. Each use-case has different requirements for workload type, scaling, storage, networking, and security. By splitting these into dedicated charts, each chart can provide sensible defaults, purpose-built configuration, and a simpler user experience for its specific use-case.

## Fluent Bit Collector

The [Fluent Bit Collector Helm Chart](https://artifacthub.io/packages/helm/fluent-bit-collector/fluent-bit-collector) deploys Fluent Bit as a **DaemonSet**, running a pod on every node in the cluster to collect logs and forward them to a backend.

Key features include:

- **Kubernetes log collection out-of-the-box** — pre-configured to tail container logs from `/var/log/containers/` with CRI multiline parsing and Kubernetes metadata enrichment via the `kubernetes` filter processor.
- **Kubernetes tag-based routing** — logs are tagged with `kube.<namespace>.<pod>.<container>`, making it easy to route logs based on namespace, pod, or container name.
- **Filesystem-backed buffering** — configurable host filesystem storage for data durability to protect against data loss during restarts or backpressure.
- **YAML-based pipeline configuration** — the chart uses Fluent Bit's native YAML configuration format, making it easier to read and maintain compared to classic INI-style configuration.
- **Hot-reload support** — an optional sidecar container watches for configuration changes and triggers a Fluent Bit hot-reload, enabling configuration updates without pod restarts.
- **Prometheus monitoring** — built-in support for creating `ServiceMonitor` or `PodMonitor` resources for the Prometheus Operator.

## Fluent Bit Aggregator

The [Fluent Bit Aggregator Helm Chart](https://artifacthub.io/packages/helm/fluent-bit-aggregator/fluent-bit-aggregator) deploys Fluent Bit as a **StatefulSet**, designed to receive logs from multiple collectors (or other sources) and forward them to a central destination. This pattern is useful for centralizing log processing, applying cross-source transformations, and managing output connections to external systems.

Key features include:

- **StatefulSet with headless service** — provides stable network identities and ordered deployment, which is important for reliable log aggregation.
- **Forward input by default** — pre-configured with a forward input plugin to receive logs from Fluent Bit collectors or Fluentd forwarders.
- **Persistent storage** — optional `PersistentVolumeClaim` support for filesystem-backed buffering, protecting against data loss when aggregator pods are restarted.
- **Horizontal Pod Autoscaling** — built-in `HorizontalPodAutoscaler` support to scale the aggregator based on resource utilisation or custom metrics.
- **Pod Disruption Budgets** — configurable `PodDisruptionBudget` to ensure availability during cluster maintenance.
- **Ingress support** — configurable ingresses for each input plugin, allowing external sources to forward logs into the aggregator.
- **Upstream servers** — support for defining upstream server groups for round-robin data distribution across output backends.
- **Grafana dashboards** — optional Grafana dashboard installation for monitoring aggregator performance, including input/output byte and record rates, error rates, and resource utilisation.

## Shared Features

Both charts share a number of common capabilities:

- **YAML-based configuration** — pipeline configuration is defined in Helm values using Fluent Bit's native YAML format, eliminating the need to manage raw configuration files.
- **Hot-reload** — optional sidecar-based hot-reload support for applying configuration changes without restarting pods.
- **Lua scripting** — Lua scripts can be defined in values and are automatically mounted for use in filter plugins.
- **OCI and non-OCI installation** — charts are published as OCI artifacts to `ghcr.io/fluent/helm-charts` (recommended) and via the traditional Helm repository at `https://fluent.github.io/helm-charts/`.
- **Cosign verification** — OCI chart releases are signed using [Cosign](https://github.com/sigstore/cosign), allowing users to verify chart integrity before installation.
- **Prometheus Operator integration** — built-in `ServiceMonitor` support for exposing Fluent Bit metrics.
- **Security-first defaults** — non-root containers, read-only root filesystems, and dropped capabilities out-of-the-box.

## Installation

### OCI (Recommended)

```shell
# Aggregator
helm upgrade --install fluent-bit-aggregator oci://ghcr.io/fluent/helm-charts/fluent-bit-aggregator

# Collector
helm upgrade --install fluent-bit-collector oci://ghcr.io/fluent/helm-charts/fluent-bit-collector
```

### Non-OCI Repository

```shell
helm repo add fluent https://fluent.github.io/helm-charts/
helm repo update

# Aggregator
helm upgrade --install fluent-bit-aggregator fluent/fluent-bit-aggregator

# Collector
helm upgrade --install fluent-bit-collector fluent/fluent-bit-collector
```

## Repository Improvements

As part of introducing these new charts, we've also made significant improvements to the [fluent/helm-charts](https://github.com/fluent/helm-charts) repository:

- **Automated release pipeline** — charts are now released automatically via a reusable workflow when the chart version is updated, with support for OCI publishing, GitHub Pages, and Artifact Hub linting.
- **PR validation** — pull requests are validated with helm-docs generation, changelog enforcement, Artifact Hub linting, and [Kubeconform](https://github.com/yannh/kubeconform) schema validation against the latest Kubernetes versions.
- **OpenSSF Scorecard** — the repository now runs the [OpenSSF Scorecard](https://securityscorecards.dev/) to track security best practices.

## Getting Started

Visit the chart pages on Artifact Hub for full documentation and default values:

- [fluent-bit-aggregator](https://artifacthub.io/packages/helm/fluent-bit-aggregator/fluent-bit-aggregator)
- [fluent-bit-collector](https://artifacthub.io/packages/helm/fluent-bit-collector/fluent-bit-collector)

If you have questions or feedback, please open an issue on the [fluent/helm-charts](https://github.com/fluent/helm-charts) repository or join the conversation on the [Fluent Slack](https://www.launchpass.com/fluent-all).
