---
name: lite-server-node-ops
description: Guide for Lite Server node operations and maintenance, including fault diagnosis, vulnerability patching, one-click stress testing, and parameter plane network configuration. This skill should be used when users need to diagnose node faults, fix security vulnerabilities, run hardware stress tests, or configure management network on Lite Server nodes.
license: Apache 2.0
---

# Lite Server Node Operations Guide

## Overview

This skill provides guidance for operating and maintaining Lite Server nodes, covering fault diagnosis, vulnerability remediation, hardware stress testing, and parameter plane (management) network configuration.

---

## When to Use

- Diagnosing hardware or software faults on Lite Server nodes
- Patching security vulnerabilities on Lite Server nodes
- Running one-click stress tests to validate node health
- Configuring parameter plane network for node management

---

## Node Fault Diagnosis

### Common Fault Types

| Fault Type | Symptom | Severity |
|---|---|---|
| NPU hardware error | npu-smi reports ECC error | Critical |
| NPU link down | Inter-chip communication failure | Critical |
| NPU overtemperature | Chip temperature exceeds threshold | High |
| Disk failure | I/O errors in system logs | High |
| Network unreachable | SSH connection lost | Medium |
| OS kernel panic | Node unresponsive | Critical |

### Diagnosis Workflow

```
1. Check node status in ModelArts Console
   └── Status: Running / Error / Unreachable

2. If accessible via SSH:
   ├── Check NPU status: npu-smi info
   ├── Check system logs: dmesg | tail -100
   ├── Check disk: df -h
   └── Check network: ping / ip addr

3. If not accessible:
   └── Use console fault diagnosis tool
```

### Diagnostic Commands

```bash
# NPU health check
npu-smi info
npu-smi info -t health

# System log check
dmesg | grep -i error
journalctl -p err --since "1 hour ago"

# Disk health
df -h
iostat -x 1 5

# Network check
ip addr show
ping -c 3 <peer_node_ip>
```

Reference: [Lite Server节点故障诊断](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0031.html)

---

## Node Vulnerability Patching

### Overview

Lite Server nodes may have OS-level or software-level security vulnerabilities that need timely remediation.

### Patching Workflow

1. ModelArts Console displays vulnerability alerts for affected nodes
2. Review vulnerability details (CVE ID, severity, affected packages)
3. Schedule a maintenance window (stop running jobs)
4. Initiate vulnerability fix from the console
5. System applies patches and may restart the node
6. Verify node status after patching

### Precautions

- Always back up data before applying patches
- Stop running training jobs to avoid data corruption
- Some patches require node restart
- Verify NPU functionality after patching

Reference: [Lite Server节点漏洞修复](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0057.html)

---

## One-Click Stress Testing

### Purpose

Stress testing validates node hardware health by running intensive workloads on NPU, CPU, memory, disk, and network to detect potential hardware issues before production use.

### Test Items

| Test Item | What It Tests | Pass Criteria |
|---|---|---|
| NPU compute | AI Core computation | No errors, expected throughput |
| NPU memory (HBM) | HBM read/write | No ECC errors |
| HCCS link | Inter-chip communication | Stable bandwidth, no packet loss |
| RoCE network | Inter-node communication | Expected bandwidth, low latency |
| CPU | Processor stability | No errors under load |
| Memory | RAM read/write | No errors |
| Disk I/O | Storage performance | Expected IOPS and throughput |

### Running Stress Test

1. Navigate to Lite Server → Node Operations
2. Select target node
3. Click "One-Click Stress Test"
4. Choose test items (all or selective)
5. Start the test and wait for results
6. Review test report for any failures

Reference: [Lite Server节点一键式压测](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0035.html)

---

## Parameter Plane Network Configuration

### What is the Parameter Plane

The parameter plane (参数面) is a dedicated management network used for:

- Node health monitoring and heartbeat
- Plugin management and updates
- Log collection and metric reporting
- Remote management operations

### Configuration

Configure the parameter plane network to ensure reliable management communication:

- Separate from the data plane (training traffic) network
- Ensure low latency and high availability
- Configure appropriate security group rules for management traffic

Reference: [Lite Server节点参数面网络配置](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0047.html)

---

## Reference Documentation

- [Lite Server节点故障诊断](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0031.html)
- [Lite Server节点漏洞修复](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0057.html)
- [Lite Server节点一键式压测](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0035.html)
- [Lite Server节点参数面网络配置](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0047.html)
