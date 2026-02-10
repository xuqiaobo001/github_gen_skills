---
name: lite-cluster-plugin
description: Guide for managing Lite Cluster plugins, including Node Agent for fault detection, Metrics Collector for monitoring, Device Plugin for NPU scheduling, Volcano scheduler, and Cluster Elastic Engine. This skill should be used when users need to install, upgrade, or troubleshoot plugins on Lite Cluster.
license: Apache 2.0
---

# Lite Cluster Plugin Management Guide

## Overview

This skill provides guidance for managing plugins on ModelArts Lite Cluster. Plugins extend cluster functionality with fault detection, metric collection, NPU device scheduling, job scheduling, and elastic scaling capabilities.

---

## When to Use

- Installing or upgrading Lite Cluster plugins
- Configuring Node Agent for node fault detection
- Setting up Metrics Collector for NPU/GPU monitoring
- Managing Device Plugin for NPU resource scheduling
- Configuring Volcano scheduler for distributed training jobs
- Enabling Cluster Elastic Engine for auto-scaling

---

## Plugin Overview

| Plugin | Purpose | Required |
|---|---|---|
| ModelArts Node Agent | Node fault detection and auto-recovery | Yes |
| ModelArts Metrics Collector | NPU/GPU/system metric collection | Yes |
| ModelArts Device Plugin | NPU device registration and scheduling | Yes |
| Volcano Scheduler | Gang scheduling for distributed training | Yes |
| Cluster Elastic Engine | Auto-scaling based on workload demand | Optional |

### Plugin Lifecycle

```
Install → Configure → Running → Upgrade → (Uninstall)
```

All plugins are managed via the ModelArts Console → Lite Cluster → Plugin Management page.

Reference: [Lite Cluster插件概述](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0034.html)

---

## ModelArts Node Agent

### What is Node Agent

ModelArts Node Agent runs as a DaemonSet on every cluster node. It monitors node health, detects hardware faults (NPU errors, link failures, disk issues), and triggers automatic fault handling.

### Fault Detection Capabilities

| Fault Type | Detection Method | Auto Action |
|---|---|---|
| NPU ECC error | Periodic npu-smi health check | Mark node as faulty, cordon |
| NPU link down | HCCS/RoCE link monitoring | Alert and cordon node |
| NPU overtemperature | Temperature threshold monitoring | Alert, throttle if critical |
| Disk failure | I/O error monitoring | Alert, mark node NotReady |
| OS hang | Heartbeat timeout | Restart or isolate node |

### Verify Node Agent

```bash
# Check Node Agent DaemonSet
kubectl get ds -n kube-system | grep node-agent

# Check Node Agent pods
kubectl get pods -n kube-system -l app=modelarts-node-agent

# View Node Agent logs
kubectl logs -n kube-system -l app=modelarts-node-agent --tail=50
```

Reference: [节点故障检测(ModelArts Node Agent)](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0035.html)

---

## ModelArts Metrics Collector

### What is Metrics Collector

ModelArts Metrics Collector runs as a DaemonSet that collects NPU, GPU, and system-level metrics from each node and exposes them for AOM and Prometheus consumption.

### Collected Metrics

| Category | Metrics |
|---|---|
| NPU | Utilization, HBM usage, temperature, power, ECC errors |
| GPU | Utilization, VRAM usage, temperature, power |
| System | CPU, memory, disk I/O, network throughput |
| Pod | Per-pod resource consumption |

### Verify Metrics Collector

```bash
# Check Metrics Collector DaemonSet
kubectl get ds -n kube-system | grep metrics-collector

# Check Metrics Collector pods
kubectl get pods -n kube-system -l app=modelarts-metrics-collector

# View exposed metrics endpoint
kubectl exec -n kube-system <metrics-collector-pod> -- curl -s localhost:9100/metrics | head -20
```

Reference: [指标监控插件(ModelArts Metrics Collector)](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0041.html)

---

## ModelArts Device Plugin (AI套件)

### What is Device Plugin

ModelArts Device Plugin implements the Kubernetes Device Plugin interface for Ascend NPU devices. It registers NPU resources with the kubelet so that pods can request NPU devices via standard K8s resource requests.

### How It Works

```
Device Plugin (DaemonSet on each node)
  ├── Discovers local NPU devices
  ├── Registers them with kubelet as extended resources
  │     └── e.g., huawei.com/ascend-910b: 8
  └── Allocates devices to pods on scheduling
```

### Resource Request in Pod Spec

```yaml
resources:
  limits:
    huawei.com/ascend-910b: "4"  # Request 4 NPU devices
```

### Verify Device Plugin

```bash
# Check Device Plugin DaemonSet
kubectl get ds -n kube-system | grep device-plugin

# Check allocatable NPU resources on nodes
kubectl get nodes -o custom-columns=NAME:.metadata.name,NPU:.status.allocatable.huawei\\.com/ascend-910b
```

Reference: [AI套件(ModelArts Device Plugin)](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0037.html)

---

## Volcano Scheduler

### What is Volcano

Volcano is a Kubernetes-native batch scheduling system designed for high-performance computing and AI workloads. It provides gang scheduling, fair-share scheduling, and queue management essential for distributed training.

### Key Features

| Feature | Description |
|---|---|
| Gang scheduling | All pods in a job start together or none start |
| Queue management | Priority-based job queues with resource quotas |
| Fair-share | Balanced resource allocation across multiple jobs |
| Preemption | Higher-priority jobs can preempt lower-priority ones |
| Topology-aware | Schedule pods considering NPU topology |

### Verify Volcano

```bash
# Check Volcano components
kubectl get pods -n volcano-system

# Check Volcano scheduler
kubectl get pods -n volcano-system -l app=volcano-scheduler

# List Volcano jobs
kubectl get vcjob -A

# List Volcano queues
kubectl get queue
```

### Queue Configuration

```yaml
apiVersion: scheduling.volcano.sh/v1beta1
kind: Queue
metadata:
  name: training-queue
spec:
  weight: 1
  capability:
    huawei.com/ascend-910b: "64"
```

Reference: [Volcano调度器](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0039.html)

---

## Cluster Elastic Engine

### What is Cluster Elastic Engine

The Cluster Elastic Engine plugin enables automatic scaling of Lite Cluster node pools based on workload demand. When pending pods cannot be scheduled due to insufficient resources, the engine triggers scale-up; when nodes are underutilized, it triggers scale-down.

### Scaling Policies

| Policy | Description |
|---|---|
| Scale-up trigger | Pending pods with unmet resource requests |
| Scale-up limit | Maximum node count per node pool |
| Scale-down trigger | Node utilization below threshold for sustained period |
| Scale-down cooldown | Minimum time between scale-down operations |
| Scale-down protection | Nodes with running non-evictable pods are protected |

### Configuration

1. Navigate to ModelArts Console → Lite Cluster → Plugin Management
2. Install or enable "Cluster Elastic Engine"
3. Configure scaling parameters:
   - Minimum and maximum node count
   - Scale-down utilization threshold (e.g., < 30% for 10 minutes)
   - Cooldown period between scaling events

### Verify Elastic Engine

```bash
# Check Elastic Engine deployment
kubectl get deploy -n kube-system | grep elastic-engine

# View scaling events
kubectl get events --field-selector reason=ScaledUp
kubectl get events --field-selector reason=ScaledDown
```

Reference: [集群弹性引擎](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0038.html)

---

## Troubleshooting

| Issue | Possible Cause | Solution |
|---|---|---|
| Plugin install fails | Network or permission issue | Check security groups and IAM permissions |
| Node Agent not detecting faults | Agent pod crashed or not running | Check DaemonSet status, restart if needed |
| Metrics not appearing in AOM | Metrics Collector not installed | Install Metrics Collector plugin |
| NPU not schedulable | Device Plugin not running | Check Device Plugin DaemonSet |
| Gang scheduling not working | Volcano not installed | Install Volcano scheduler plugin |
| Auto-scaling not triggering | Elastic Engine not enabled | Enable and configure Elastic Engine |

---

## Reference Documentation

- [Lite Cluster插件管理](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0033.html)
- [Lite Cluster插件概述](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0034.html)
- [节点故障检测(ModelArts Node Agent)](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0035.html)
- [指标监控插件(ModelArts Metrics Collector)](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0041.html)
- [AI套件(ModelArts Device Plugin)](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0037.html)
- [Volcano调度器](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0039.html)
- [集群弹性引擎](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0038.html)
