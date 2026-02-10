---
name: lite-cluster-quickstart
description: Guide for getting started with ModelArts Lite Cluster, including usage flow, high-risk operations, software version compatibility, and resource provisioning. This skill should be used when users need to understand Lite Cluster prerequisites, plan resource provisioning, or check software compatibility for different hardware models.
license: Apache 2.0
---

# Lite Cluster Quick Start Guide

## Overview

This skill provides guidance for getting started with ModelArts Lite Cluster — a Kubernetes-based managed cluster service for large-scale AI training and inference on Ascend NPU hardware. It covers the usage flow, high-risk operation awareness, software compatibility, and resource provisioning.

---

## When to Use

- Understanding Lite Cluster concepts and usage flow
- Planning Lite Cluster resource provisioning
- Checking software version compatibility for specific hardware models
- Reviewing high-risk operations before performing cluster management tasks

---

## Lite Cluster Overview

### What is Lite Cluster

Lite Cluster is a Kubernetes-based managed compute cluster provided by ModelArts for large-scale distributed AI training and inference. Unlike Lite Server (bare-metal single instances), Lite Cluster provides:

| Feature | Description |
|---|---|
| Kubernetes orchestration | Native K8s API, kubectl access, pod scheduling |
| Node pools | Logical grouping of nodes by hardware specification |
| Volcano scheduler | AI-optimized job scheduling for distributed training |
| Plugin ecosystem | Extensible with Node Agent, Device Plugin, Metrics Collector |
| Elastic scaling | Dynamic scale-up/down of node pools |

### Lite Cluster vs Lite Server

| Aspect | Lite Cluster | Lite Server |
|---|---|---|
| Management model | Kubernetes-based | Bare-metal instance |
| Access method | kubectl / K8s API | SSH |
| Job scheduling | Volcano scheduler | Manual |
| Scaling | Elastic node pool scaling | Manual provisioning |
| Use case | Large-scale distributed training | Single-node or small-scale tasks |

Reference: [ModelArts Lite Cluster用户指南](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0001.html)

---

## Usage Flow

### End-to-End Workflow

```
1. Resource Provisioning (资源开通)
   └── Apply for Lite Cluster resource pool

2. Resource Configuration (资源配置)
   ├── Configure network (VPC, subnet, security groups)
   ├── Configure kubectl access
   ├── Configure storage (SFS Turbo, OBS)
   ├── (Optional) Configure driver version
   └── (Optional) Configure image pre-warming

3. Resource Usage (资源使用)
   ├── Submit distributed training jobs
   ├── Run inference workloads
   └── Use diagnostic tools

4. Resource Management (资源管理)
   ├── Manage resource pools, node pools, nodes
   ├── Scale up/down
   └── Upgrade drivers

5. Monitoring & Release (监控与释放)
   ├── Monitor via AOM / Prometheus
   └── Release resources when no longer needed
```

Reference: [Lite Cluster使用流程](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0002.html)

---

## High-Risk Operations

### Critical Operations Requiring Caution

| Operation | Risk Level | Impact | Precaution |
|---|---|---|---|
| Resource pool deletion | Critical | All data and workloads lost | Back up all data first |
| Node pool scaling down | High | Running pods evicted | Drain nodes before removal |
| Driver upgrade | High | Node restart required | Stop jobs, back up data |
| Network reconfiguration | High | Connectivity disruption | Plan maintenance window |
| Plugin uninstallation | Medium | Feature loss | Verify no dependent workloads |

### General Precautions

- Always back up data before destructive operations
- Stop running training/inference jobs before maintenance
- Test changes on a single node before applying cluster-wide
- Review the high-risk operations checklist before each operation

Reference: [Lite Cluster高危操作一览表](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0003.html)

---

## Software Version Compatibility

### Version Compatibility Chain

```
Hardware Model (e.g., Snt9B / Atlas 900A)
  └── Driver version
       └── Firmware version
            └── CANN version
                 └── torch_npu version
                      └── Container image version
```

All components in the chain must be version-compatible. Mismatched versions cause runtime errors or training failures.

### Checking Compatibility

1. Identify your hardware model from the resource pool details
2. Refer to the compatibility matrix for supported driver/CANN/framework versions
3. Ensure container images match the installed driver and CANN versions

Reference: [不同机型对应的软件配套版本](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0004.html)

---

## Resource Provisioning

### Prerequisites

1. Huawei Cloud account with ModelArts service enabled
2. Sufficient quota for the desired hardware specification
3. VPC and subnet pre-created in the target region
4. IAM permissions for ModelArts Lite Cluster operations

### Provisioning Steps

1. Navigate to ModelArts Console → Lite Cluster
2. Click "Apply for Resource Pool"
3. Select hardware specification (e.g., Snt9B)
4. Specify the number of nodes
5. Configure billing mode (yearly/monthly)
6. Submit the application and wait for approval and provisioning

Reference: [Lite Cluster资源开通](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0005.html)

---

## Reference Documentation

- [ModelArts Lite Cluster用户指南](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0001.html)
- [Lite Cluster使用流程](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0002.html)
- [Lite Cluster高危操作一览表](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0003.html)
- [不同机型对应的软件配套版本](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0004.html)
- [Lite Cluster资源开通](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0005.html)
