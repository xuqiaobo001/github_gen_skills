---
name: lite-server-supernode
description: Guide for managing Lite Server super nodes (超节点), including scaling, system disk expansion, periodic stress testing, and HCCL operator-level re-execution. This skill should be used when users need to manage multi-server super node clusters for large-scale distributed training on Lite Server.
license: Apache 2.0
---

# Lite Server Super Node Management Guide

## Overview

This skill provides guidance for managing Lite Server super nodes (超节点) — clusters of multiple Lite Server instances grouped for large-scale distributed training. It covers super node scaling, disk expansion, periodic stress testing, and HCCL communication reliability configuration.

---

## When to Use

- Scaling super node clusters up or down
- Expanding system disk capacity on super nodes
- Running periodic stress tests on super node clusters
- Enabling HCCL operator-level re-execution for training reliability
- Managing multi-node training infrastructure

---

## Super Node Overview

A super node (超节点) is a logical grouping of multiple Lite Server instances that work together as a unified compute cluster for large-scale distributed training.

### Key Characteristics

| Feature | Description |
|---|---|
| Composition | Multiple Lite Server instances with same specification |
| Network | High-speed HCCS/RoCE interconnect between nodes |
| Scheduling | Unified job scheduling across all nodes |
| Scaling | Dynamic add/remove nodes |

Reference: [Lite Server超节点管理](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0041.html)

---

## Super Node Scaling

### Scale Up (扩容)

Add new nodes to an existing super node cluster:

1. Navigate to Lite Server → Super Node Management
2. Select the target super node
3. Click "Scale Up"
4. Specify the number of nodes to add
5. New nodes must use the same compute specification
6. Wait for provisioning and network configuration to complete

### Scale Down (缩容)

Remove nodes from a super node cluster:

1. Ensure no running jobs on the nodes to be removed
2. Select nodes to remove
3. Confirm scale-down operation
4. System gracefully drains and removes the nodes

### Precautions

- New nodes are automatically configured with the same network and software environment
- Scale-down will fail if running jobs exist on target nodes
- Minimum 1 node must remain in the super node

Reference: [Lite Server超节点扩容和缩容](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0040.html)

---

## System Disk Expansion

When the system disk on super node instances runs out of space:

1. Navigate to Super Node Management → select target node
2. Click "Expand System Disk"
3. Specify the new disk size (must be larger than current)
4. Confirm and wait for expansion to complete
5. Node may require restart

**Note**: Expansion only increases disk size; shrinking is not supported.

Reference: [Lite Server超节点系统盘扩容](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0055.html)

---

## Periodic Stress Testing

### Purpose

Periodic stress testing on super nodes proactively detects hardware degradation before it causes training failures. Recommended to run regularly (e.g., weekly or before major training jobs).

### Test Scope

| Component | Test Method |
|---|---|
| NPU compute | Run standard AI benchmark workloads |
| HCCS links | Bandwidth and latency test between NPU chips |
| RoCE network | Cross-node communication bandwidth test |
| Storage I/O | Sequential and random read/write tests |

### Scheduling

1. Navigate to Super Node Management
2. Select "Periodic Stress Test"
3. Configure schedule (e.g., every Sunday 02:00)
4. Select test items
5. Enable notification for test failures

Reference: [Lite Server超节点定期压测](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0036.html)

---

## HCCL Operator-Level Re-execution

### What is HCCL Re-execution

HCCL (Huawei Collective Communication Library) operator-level re-execution is a reliability mechanism that automatically retries failed communication operators during distributed training, instead of failing the entire job.

### Benefits

- Tolerates transient network glitches without job failure
- Reduces training interruption caused by intermittent HCCS/RoCE errors
- Improves overall training completion rate for long-running jobs

### How to Enable

1. Navigate to Super Node Management
2. Select "HCCL Configuration"
3. Enable "Operator-Level Re-execution"
4. Configure retry parameters:

| Parameter | Description | Recommended Value |
|---|---|---|
| Max retries | Maximum retry attempts per operator | 3 |
| Retry timeout | Timeout for each retry (seconds) | 300 |

Reference: [开启超节点HCCL通信算子级重执行机制](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0037.html)

---

## Reference Documentation

- [Lite Server超节点管理](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0041.html)
- [Lite Server超节点扩容和缩容](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0040.html)
- [Lite Server超节点系统盘扩容](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0055.html)
- [Lite Server超节点定期压测](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0036.html)
- [开启超节点HCCL通信算子级重执行机制](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0037.html)
