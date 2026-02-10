---
name: ascend-monitoring
description: Guide for monitoring Ascend NPU resource utilization and configuring alerts on ModelArts Standard. This skill should be used when users need to set up NPU monitoring dashboards with Grafana, configure AOM/CES alert rules, collect NPU metrics like utilization and HBM usage, or analyze audit logs for Ascend workloads.
license: Apache 2.0
---

# Ascend Resource Monitoring Guide

## Overview

This skill provides guidance for monitoring Huawei Ascend NPU resources on ModelArts Standard. It covers NPU metric collection, Grafana dashboard setup, alert configuration via AOM and CES, and audit log analysis.

---

## When to Use

- Setting up NPU utilization monitoring dashboards
- Configuring Grafana to visualize Ascend NPU metrics
- Setting up alert rules for NPU resource anomalies
- Collecting and analyzing NPU metrics (utilization, HBM, temperature)
- Reviewing audit logs for ModelArts operations

---

## NPU Metrics Overview

### Key Ascend NPU Metrics

| Metric | Description | Unit | Normal Range |
|---|---|---|---|
| NPU utilization | AI Core compute usage | % | 70-100% (training) |
| HBM usage | High Bandwidth Memory used | MB/GB | 60-95% |
| HBM total | Total HBM capacity | GB | Fixed per model |
| NPU temperature | Chip temperature | Celsius | < 85 |
| NPU power | Power consumption | Watts | Varies by model |
| HCCS bandwidth | Inter-chip link bandwidth | GB/s | Stable |
| RoCE bandwidth | Inter-node network bandwidth | GB/s | Stable |

### Collecting Metrics via npu-smi

```bash
# Basic NPU status
npu-smi info

# Detailed per-chip info
npu-smi info -t usages -i 0

# Continuous monitoring (refresh every 1 second)
watch -n 1 npu-smi info

# JSON output for programmatic parsing
npu-smi info -t board -f json
```

---

## Monitoring Platforms

### Platform 1: ModelArts Console Built-in Monitoring

The simplest approach — view metrics directly in the ModelArts console:

- Navigate to resource pool details page
- View per-node NPU utilization, HBM usage, CPU, memory
- View per-job resource consumption

Reference: [在ModelArts控制台查看监控指标](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0060.html)

### Platform 2: AOM (Application Operations Management)

AOM provides centralized metric storage and querying:

- All ModelArts NPU metrics are automatically reported to AOM
- Supports custom metric queries via PromQL
- Supports alert rule configuration

Reference: [在AOM控制台查看ModelArts所有监控指标](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0033.html)

### Platform 3: Grafana + AOM

For advanced visualization, connect Grafana to AOM as a data source.

#### Grafana Installation Options

| Environment | Method | Reference |
|---|---|---|
| Windows | Download installer from grafana.com | [Windows安装](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0036.html) |
| Linux | `apt-get install grafana` or binary | [Linux安装](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0037.html) |
| Notebook | Install within ModelArts Notebook | [Notebook安装](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0038.html) |

#### Configure AOM as Grafana Data Source

1. Open Grafana → Configuration → Data Sources → Add data source
2. Select "Prometheus" type
3. Set URL to AOM Prometheus endpoint
4. Configure authentication with AK/SK
5. Test and save the data source

Reference: [配置Grafana数据源](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0039.html)

#### Recommended Dashboard Panels

| Panel | PromQL Query Example | Purpose |
|---|---|---|
| NPU Utilization | `npu_utilization{node="$node"}` | Per-node NPU usage |
| HBM Usage | `npu_hbm_used{node="$node"}` | Memory consumption |
| NPU Temperature | `npu_temperature{node="$node"}` | Thermal monitoring |
| Training Throughput | `training_samples_per_second` | Training speed |

Reference: [配置仪表盘查看指标数据](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0040.html)

---

## Alert Configuration

### CES (Cloud Eye Service) Event Monitoring

CES monitors ModelArts events and triggers alerts:

| Event | Severity | Description |
|---|---|---|
| Node fault detected | Critical | NPU node hardware failure |
| Training job failed | Major | Training job exited with error |
| Resource pool scaling failed | Major | Scale-up/down operation failed |
| Driver upgrade completed | Info | NPU driver upgrade finished |

Reference: [ModelArts支持的事件监控的事件说明](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0043.html)

### Recommended Alert Rules

| Metric | Condition | Action |
|---|---|---|
| NPU utilization | < 10% for > 30 min | Notify — possible idle resource waste |
| NPU utilization | < 50% for > 10 min | Warn — possible training hang |
| HBM usage | > 95% | Warn — risk of OOM |
| NPU temperature | > 85 C | Critical — throttling imminent |
| Training loss | NaN detected | Critical — training diverged |

Reference: [在CES监控ModelArts](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0041.html)

---

## Audit Logs

### CTS (Cloud Trace Service) Integration

ModelArts Standard operations are automatically recorded by CTS for compliance and troubleshooting:

- Resource pool creation/deletion/scaling
- Training job submission/stop
- Model deployment/update
- Service access and authentication events

### Viewing Audit Logs

1. Navigate to CTS console
2. Filter by service: "ModelArts"
3. Filter by resource type and time range
4. View operation details including operator, timestamp, and result

Reference: [查看ModelArts Standard相关审计日志](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0016.html)

---

## Reference Documentation

- [ModelArts Standard资源监控概述](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0032.html)
- [使用Grafana查看AOM中的监控指标](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0034.html)
- [使用CTS审计ModelArts Standard服务](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0014.html)
- [ModelArts Standard支持云审计的关键操作](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0015.html)
