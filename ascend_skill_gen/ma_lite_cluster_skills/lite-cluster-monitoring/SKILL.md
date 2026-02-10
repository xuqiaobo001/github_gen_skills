---
name: lite-cluster-monitoring
description: Guide for monitoring Lite Cluster resources using AOM and Prometheus, and releasing Lite Cluster resources. This skill should be used when users need to set up monitoring dashboards, view cluster metrics, or release Lite Cluster resources when no longer needed.
license: Apache 2.0
---

# Lite Cluster Monitoring & Resource Release Guide

## Overview

This skill provides guidance for monitoring ModelArts Lite Cluster resources using AOM (Application Operations Management) and Prometheus, as well as releasing cluster resources when they are no longer needed.

---

## When to Use

- Viewing Lite Cluster monitoring metrics via AOM
- Setting up Prometheus-based monitoring for Lite Cluster
- Building custom monitoring dashboards for NPU/GPU metrics
- Releasing Lite Cluster resources to stop billing

---

## AOM Monitoring

### What is AOM

AOM (Application Operations Management) is Huawei Cloud's built-in monitoring service that collects and displays metrics from Lite Cluster nodes and workloads without additional setup.

### Key Metrics

| Category | Metric | Description | Unit |
|---|---|---|---|
| NPU | NPU utilization | AI Core compute usage | % |
| NPU | HBM usage | High Bandwidth Memory used | MB/GB |
| NPU | NPU temperature | Chip temperature | Celsius |
| Node | CPU utilization | CPU usage per node | % |
| Node | Memory usage | RAM usage per node | % |
| Node | Disk I/O | Disk read/write throughput | MB/s |
| Network | Network throughput | Inbound/outbound traffic | MB/s |

### Viewing Metrics

1. Navigate to AOM Console → Monitoring → Cloud Service Monitoring
2. Select "ModelArts" service
3. Filter by Lite Cluster resource pool
4. Select target nodes or pods
5. Choose metrics and time range
6. Create custom dashboards for frequently viewed metrics

Reference: [使用AOM查看Lite Cluster监控指标](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0025.html)

---

## Prometheus Monitoring

### Overview

Prometheus provides flexible, self-managed monitoring for Lite Cluster with custom metric collection, alerting rules, and Grafana dashboard integration.

### Architecture

```
Lite Cluster Nodes
  ├── ModelArts Metrics Collector (exports NPU/GPU metrics)
  └── Node Exporter (exports system metrics)
        │
        ▼
  Prometheus Server (scrapes metrics)
        │
        ▼
  Grafana (visualization dashboards)
```

### Setup Steps

1. **Deploy Prometheus**: Install Prometheus server in the cluster or use an external instance

```bash
# Example: Deploy Prometheus via Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus -n monitoring --create-namespace
```

2. **Configure scrape targets**: Add Lite Cluster metric endpoints to Prometheus config

```yaml
scrape_configs:
  - job_name: 'npu-metrics'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_label_app]
        regex: metrics-collector
        action: keep
```

3. **Set up Grafana dashboards**: Import or create dashboards for NPU utilization, HBM usage, temperature, and training throughput

### Key Prometheus Metrics

| Metric Name | Description |
|---|---|
| `npu_utilization` | NPU AI Core utilization percentage |
| `npu_hbm_used` | HBM memory used in bytes |
| `npu_temperature` | NPU chip temperature in Celsius |
| `npu_power` | NPU power consumption in Watts |

Reference: [使用Prometheus查看Lite Cluster监控指标](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0026.html)

---

## Resource Release

### Overview

When Lite Cluster resources are no longer needed, release them to stop billing and free up quota.

### Pre-Release Checklist

1. **Back up all data** — training checkpoints, datasets, configuration files
2. **Export logs** — collect and save important training and system logs
3. **Stop all running jobs** — ensure no active training or inference workloads
4. **Remove PVCs** — delete PersistentVolumeClaims to detach storage
5. **Record configuration** — save network, plugin, and driver settings for future reference

### Release Steps

1. Navigate to ModelArts Console → Lite Cluster → Resource Pool
2. Select the target resource pool
3. Click "Release Resources"
4. Confirm the release operation
5. System releases all associated nodes and network resources

**Warning**: Resource release is irreversible. All data on cluster nodes will be permanently deleted.

Reference: [释放Lite Cluster资源](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0027.html)

---

## Reference Documentation

- [监控Lite Cluster资源](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0024.html)
- [使用AOM查看Lite Cluster监控指标](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0025.html)
- [使用Prometheus查看Lite Cluster监控指标](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0026.html)
- [释放Lite Cluster资源](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0027.html)
