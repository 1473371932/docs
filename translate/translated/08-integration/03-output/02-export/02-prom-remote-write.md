---
title: Prometheus Remote Write
permalink: /integration/output/export/prometheus-remote-write
---

> This document was translated by ChatGPT

# Functionality

Using Prometheus Remote Write, you can export metrics generated by DeepFlow to external platforms. This allows you to continue leveraging the Prometheus ecosystem, such as viewing metrics and configuring alerts through Prometheus.

# Metrics Overview

Within DeepFlow, metrics can be categorized into two types:

- Application Performance Metrics: [Refer to details](../../../features/universal-map/application-metrics/)
  - Corresponds to `flow_metrics.application*` table data in ClickHouse
- Network Performance Metrics: [Refer to details](../../../features/universal-map/network-metrics/)
  - Corresponds to `flow_metrics.network*` table data in ClickHouse

# Prometheus Remote Write

For protocol format, refer to Prometheus's pb file definition: https://github.com/prometheus/prometheus/blob/main/prompb/remote.proto

# DeepFlow Server Configuration Guide

Add the following configuration under the Server settings to enable metric export:

```yaml
ingester:
  exporters:
    - protocol: prometheus
      enabled: true
      endpoints: [http://127.0.0.1:9091/receive, http://1.1.1.1:9091/receive]
      data-sources:
        - flow_metrics.application_map.1s
      # - flow_metrics.application_map.1m
      # - flow_metrics.application.1s
      # - flow_metrics.application.1m
      # - flow_metrics.network_map.1s
      # - flow_metrics.network_map.1m
      # - flow_metrics.network.1s
      # - flow_metrics.network.1m
      queue-count: 4
      queue-size: 100000
      batch-size: 1024
      flush-timeout: 10
      tag-filters:
      export-fields:
        - $tag
        - $metrics
      extra-headers:
        key1: value1
        key2: value2
      export-empty-tag: false
      export-empty-metrics-disabled: false
      enum-translate-to-name-disabled: false
      universal-tag-translate-to-name-disabled: false
```

# Detailed Parameter Description

| Field          | Type    | Required | Description                                                             |
| -------------- | ------- | -------- | ----------------------------------------------------------------------- |
| protocol       | string  | Yes      | Fixed value `prometheus`                                                |
| data-sources   | string  | Yes      | Values from `flow_metrics.*` data, does not support `flow_log.*` data   |
| endpoints      | string  | Yes      | Remote receiving address, remote write receiving address, randomly selects one that can send successfully |
| batch-size     | int     | No       | Batch size, sends in batches when this value is reached. Default: 1024  |
| extra-headers  | map     | No       | Header fields for remote HTTP requests, such as tokens for authentication |
| export-fields  | string  | Yes      | Currently does not support `$k8s.label`, recommended configuration: [$tag, $metrics] |

[Refer to detailed configuration](./exporter-config/)

# Quick Practice Demo

- Set up a RemoteWrite receiver, refer to this Prometheus [demo](https://github.com/prometheus/prometheus/tree/main/documentation/examples/remote_storage/example_write_adapter)

- Add configuration

```yaml
exporters:
  - protocol: prometheus
    data-sources:
      - flow_metrics.application_map.1s
    endpoints: [http://localhost:1234/receive]
    export-fields:
      - $tag
      - $metrics
```

- Restart DeepFlow Server, and after a short wait, you should see the output results at the RemoteWrite receiver as shown in the image

![](./imgs/remote-write.png)