---
name: ascend-resource-pool
description: Guide for managing Ascend NPU dedicated resource pools on ModelArts Standard. This skill should be used when users need to create, scale, upgrade, or troubleshoot Standard dedicated resource pools with Ascend NPU nodes, manage plugins like Volcano Scheduler and Device Plugin, or handle node fault repair.
license: Apache 2.0
---

# Ascend Resource Pool Management Guide

## Overview

This skill provides guidance for creating and managing dedicated Ascend NPU resource pools on ModelArts Standard. It covers resource pool lifecycle management, driver upgrades, node fault repair, plugin management, and logical sub-pool configuration.

---

## When to Use

- Creating a new Standard dedicated resource pool with Ascend NPU nodes
- Scaling resource pools up or down
- Upgrading NPU drivers in a resource pool
- Repairing faulty NPU nodes
- Managing resource pool plugins (Volcano, Device Plugin, etc.)
- Configuring logical sub-pools for multi-team resource sharing

---

## Resource Pool Lifecycle

### Creating a Resource Pool

Key parameters when creating an Ascend NPU resource pool:

| Parameter | Description | Example |
|---|---|---|
| Pool name | Unique identifier | `ascend-training-pool` |
| Node flavor | NPU hardware specification | `modelarts.vm.cpu168.npu.8xAscend910B` |
| Node count | Number of compute nodes | 4 |
| Network | VPC and subnet configuration | Select existing VPC |
| Job types | Supported workload types | Training, Notebook, Inference |

Important considerations:
- Select the correct NPU model (910A/910B/910C) based on workload requirements
- Ensure the VPC has sufficient IP addresses for all nodes
- Configure public network access if training requires external data download

Reference: [创建Standard专属资源池](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0004.html)

### Scaling a Resource Pool

```
Scale-up:  Add nodes to handle increased workload
Scale-down: Remove idle nodes to reduce cost
```

Scaling constraints:
- Minimum 1 node must remain in the pool
- New nodes must use the same flavor as existing nodes
- Scale-down will wait for running jobs to complete (graceful drain)

Reference: [扩缩容Standard专属资源池](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0006.html)

### Deleting a Resource Pool

Prerequisites before deletion:
1. Stop all running jobs (training, inference, notebook)
2. Remove all logical sub-pools
3. Disassociate network from SWR enterprise registry (if configured)

Reference: [删除Standard专属资源池和网络](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0010.html)

---

## Driver and Firmware Upgrades

### Version Compatibility Check

Before upgrading, verify the target version compatibility:

```
NPU Hardware → Driver → Firmware → CANN → Framework (PyTorch/MindSpore)
```

All components in the chain must be compatible. Check the official compatibility matrix.

Reference: [不同机型对应的软件配套版本](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0085.html)

### Upgrade Process

1. Navigate to resource pool details page
2. Select "Driver Upgrade" action
3. Choose target driver version from the dropdown
4. Upgrade proceeds in rolling fashion (node by node)
5. Running jobs are drained before each node upgrade

Reference: [升级Standard专属资源池驱动](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0009.html)

---

## Node Fault Repair

### Common NPU Node Faults

| Fault Type | Symptom | Auto-detected |
|---|---|---|
| NPU ECC error | Training crash or hang | Yes (via Node Agent) |
| NPU link down | Communication failure | Yes |
| NPU overtemperature | Throttling or shutdown | Yes |
| OS kernel panic | Node unreachable | Yes |
| Disk full | Job submission failure | Yes |

### Repair Workflow

1. ModelArts Node Agent detects fault and marks node as unhealthy
2. Running jobs on the faulty node are rescheduled to healthy nodes
3. Administrator initiates node repair from the console
4. System replaces or restores the faulty node
5. Node rejoins the resource pool after health check passes

Reference: [修复Standard专属资源池故障节点](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0059.html)

---

## Plugin Management

### Essential Plugins

| Plugin | Purpose | Required |
|---|---|---|
| ModelArts Node Agent | Node health monitoring and fault detection | Yes |
| ModelArts Device Plugin | NPU device scheduling and allocation | Yes |
| Volcano Scheduler | Gang scheduling for distributed training | Recommended |
| ModelArts Metric Collector | NPU metrics collection | Recommended |
| NodeLocal DNSCache | DNS resolution acceleration | Optional |
| kube-prometheus-stack | Prometheus + Grafana monitoring | Optional |

### Volcano Scheduler Configuration

Volcano is critical for distributed training — it ensures all nodes of a multi-node job start simultaneously (gang scheduling):

- Prevents deadlock where partial nodes start and wait for remaining nodes
- Supports job priority and fair-share scheduling
- Supports queue-based resource management

Reference: [Volcano调度器（Volcano Scheduler）](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0071.html)

---

## Logical Sub-pools

For multi-team environments, divide a physical resource pool into logical sub-pools:

- Each sub-pool has guaranteed resource quotas
- Teams can only use resources within their sub-pool
- Prevents resource contention between teams

Reference: [管理Standard专属资源池的逻辑子池](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0079.html)

---

## Reference Documentation

- [Standard资源池功能介绍](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0003.html)
- [管理Standard专属资源池](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0001.html)
- [Standard专属资源池插件概述](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0065.html)
- [使用TMS标签实现资源分组管理](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0063.html)
