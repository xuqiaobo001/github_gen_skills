---
name: lite-server-monitoring
description: Guide for Lite Server log collection, monitoring, and alerting, including NPU/GPU log upload, CES-based NPU resource and event monitoring, CES alerting configuration, and DCGM GPU monitoring. This skill should be used when users need to collect logs, set up monitoring dashboards, or configure alerts for Lite Server instances.
license: Apache 2.0
---

# Lite Server Monitoring & Logging Guide

## Overview

This skill provides guidance for monitoring and logging on ModelArts Lite Server, covering NPU and GPU log collection, CES (Cloud Eye Service) based resource and event monitoring, alert configuration, and DCGM-based GPU monitoring.

---

## When to Use

- Collecting and uploading NPU or GPU logs for troubleshooting
- Setting up CES monitoring for NPU resource utilization
- Monitoring NPU events via CES
- Configuring monitoring alerts and notifications
- Using DCGM to monitor GPU resources

---

## Log Collection

### NPU Log Collection and Upload

Collect NPU-related logs for troubleshooting training failures or hardware issues:

#### Log Types

| Log Type | Location | Content |
|---|---|---|
| NPU driver log | `/var/log/npu/` | Driver-level errors and events |
| HCCL log | `$ASCEND_LOG_PATH/hccl/` | Communication library logs |
| CANN runtime log | `$ASCEND_LOG_PATH/runtime/` | Operator execution logs |
| npu-smi log | Manual collection | Device status snapshots |

#### Collection Steps

1. Navigate to Lite Server → Log Collection
2. Select target server and log type
3. Specify time range for log collection
4. Click "Collect and Upload"
5. Logs are uploaded to OBS for download and analysis

#### Manual Collection Commands

```bash
# Collect NPU device info
npu-smi info > /tmp/npu_info.log

# Collect driver logs
cp -r /var/log/npu/ /tmp/npu_driver_logs/

# Collect HCCL logs (if ASCEND_LOG_PATH is set)
cp -r ${ASCEND_LOG_PATH}/hccl/ /tmp/hccl_logs/
```

Reference: [NPU日志收集上传](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0024.html)

### GPU Log Collection and Upload

For GPU-based Lite Server instances:

```bash
# Collect GPU device info
nvidia-smi > /tmp/gpu_info.log

# Collect GPU detailed logs
nvidia-smi -q > /tmp/gpu_detail.log

# Collect CUDA error logs
cat /var/log/cuda-installer.log > /tmp/cuda_install.log
```

Upload GPU logs via the console or manually to OBS for analysis.

Reference: [GPU日志收集上传](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0026.html)

---

## CES NPU Resource Monitoring

### Overview

CES (Cloud Eye Service) provides real-time monitoring of NPU resource utilization on Lite Server instances.

### Key NPU Metrics

| Metric | Description | Unit | Normal Range |
|---|---|---|---|
| NPU utilization | AI Core compute usage | % | 70-100% (training) |
| HBM usage | High Bandwidth Memory used | MB/GB | 60-95% |
| NPU temperature | Chip temperature | Celsius | < 85 |
| NPU power | Power consumption | Watts | Varies by model |

### Setup Steps

1. Ensure CES Agent is installed on the Lite Server (see Plugin Management)
2. Navigate to CES Console → Cloud Service Monitoring
3. Select "ModelArts" service
4. View NPU metrics for target Lite Server instances
5. Create custom dashboards for frequently monitored metrics

Reference: [使用CES监控Lite Server NPU资源](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0022.html)

---

## CES NPU Event Monitoring

### Monitored Events

| Event | Severity | Description |
|---|---|---|
| NPU ECC error | Critical | Memory error detected on NPU |
| NPU link down | Critical | Inter-chip communication link failure |
| NPU overtemperature | High | Chip temperature exceeds safe threshold |
| NPU device lost | Critical | NPU device no longer detected |
| Driver error | High | NPU driver encountered an error |

### Viewing Events

1. Navigate to CES Console → Event Monitoring
2. Filter by service: "ModelArts"
3. Filter by event type and severity
4. View event details including timestamp, affected node, and description

Reference: [使用CES监控Lite Server NPU事件](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0062.html)

---

## CES Alert Configuration

### Recommended Alert Rules

| Metric / Event | Condition | Severity | Action |
|---|---|---|---|
| NPU utilization | < 10% for > 30 min | Warning | Possible idle resource waste |
| NPU utilization | < 50% for > 10 min | Warning | Possible training hang |
| HBM usage | > 95% | Major | Risk of OOM |
| NPU temperature | > 85°C | Critical | Throttling imminent |
| NPU ECC error | Any occurrence | Critical | Hardware fault, schedule repair |

### Setup Steps

1. Navigate to CES Console → Alarm Rules
2. Click "Create Alarm Rule"
3. Select resource type: ModelArts Lite Server
4. Configure metric, threshold, and duration
5. Set notification channel (SMN topic, email, SMS)
6. Save and enable the rule

Reference: [使用CES实现Lite Server监控和事件告警](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0034.html)

---

## DCGM GPU Monitoring

### What is DCGM

DCGM (Data Center GPU Manager) is NVIDIA's tool for managing and monitoring GPU resources in data center environments.

### Key GPU Metrics via DCGM

| Metric | Description | Unit |
|---|---|---|
| GPU utilization | GPU compute core usage | % |
| GPU memory usage | GPU memory (VRAM) used | MB/GB |
| GPU temperature | Chip temperature | Celsius |
| GPU power | Power consumption | Watts |
| PCIe throughput | PCIe bus bandwidth | GB/s |

### Setup Steps

1. Install DCGM on GPU-based Lite Server:

```bash
# Install DCGM
apt-get install -y datacenter-gpu-manager

# Start DCGM service
systemctl start nvidia-dcgm
systemctl enable nvidia-dcgm
```

2. Collect metrics:

```bash
# List all GPUs
dcgmi discovery -l

# Monitor GPU metrics
dcgmi dmon -e 155,150,203,204
# 155=GPU utilization, 150=memory usage, 203=temperature, 204=power
```

3. Integrate with Prometheus (optional) for dashboard visualization

Reference: [使用DCGM监控Lite Server GPU资源](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0023.html)

---

## Reference Documentation

- [Lite Server日志采集](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0059.html)
- [NPU日志收集上传](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0024.html)
- [GPU日志收集上传](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0026.html)
- [Lite Server监控告警](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0021.html)
- [使用CES监控Lite Server NPU资源](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0022.html)
- [使用CES监控Lite Server NPU事件](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0062.html)
- [使用CES实现Lite Server监控和事件告警](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0034.html)
- [使用DCGM监控Lite Server GPU资源](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0023.html)
