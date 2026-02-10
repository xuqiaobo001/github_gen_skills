---
name: lite-cluster-resource-management
description: Guide for managing Lite Cluster resources, including resource pool management, node pool management, node management, elastic scaling, and driver upgrades. This skill should be used when users need to manage, scale, or upgrade Lite Cluster resource pools, node pools, or individual nodes.
license: Apache 2.0
---

# Lite Cluster Resource Management Guide

## Overview

This skill provides guidance for managing ModelArts Lite Cluster resources at three levels: resource pools, node pools, and individual nodes. It covers lifecycle management, elastic scaling, and driver upgrades.

---

## When to Use

- Managing Lite Cluster resource pool lifecycle
- Managing node pools within a resource pool
- Managing individual cluster nodes
- Scaling resource pools up or down
- Upgrading drivers on the entire pool or individual nodes

---

## Resource Management Hierarchy

```
Resource Pool (资源池)
  ├── Node Pool A (节点池) — e.g., Snt9B training nodes
  │     ├── Node-1
  │     ├── Node-2
  │     └── Node-3
  └── Node Pool B (节点池) — e.g., inference nodes
        ├── Node-4
        └── Node-5
```

| Level | Scope | Key Operations |
|---|---|---|
| Resource Pool | Entire cluster | View status, scale, upgrade drivers |
| Node Pool | Group of same-spec nodes | Add/remove nodes, configure labels |
| Node | Individual machine | View status, drain, cordon, reset |

Reference: [Lite Cluster资源管理介绍](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0018.html)

---

## Resource Pool Management

### Key Operations

| Operation | Description |
|---|---|
| View pool status | Check overall pool health, node count, resource utilization |
| View pool details | Hardware spec, driver version, network configuration |
| Modify pool | Update pool name, description, tags |

### Viewing Resource Pool

```bash
# Via kubectl — check all nodes in the pool
kubectl get nodes -o wide

# Check node labels to identify pool membership
kubectl get nodes --show-labels
```

### Console Operations

1. Navigate to ModelArts Console → Lite Cluster → Resource Pool
2. View pool overview: total nodes, healthy nodes, faulty nodes
3. Click pool name for detailed information

Reference: [管理Lite Cluster资源池](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0019.html)

---

## Node Pool Management

### Overview

A node pool groups nodes with the same hardware specification and configuration. Operations on a node pool affect all member nodes.

### Key Operations

| Operation | Description |
|---|---|
| View node pool | List nodes, check status, view spec |
| Add nodes | Add new nodes to the pool |
| Remove nodes | Drain and remove nodes from the pool |
| Configure labels | Set K8s labels/taints for scheduling |

### kubectl Operations

```bash
# List nodes with labels showing node pool
kubectl get nodes -l node-pool=<pool-name>

# Add label to node pool nodes
kubectl label nodes -l node-pool=<pool-name> workload-type=training

# Taint nodes for dedicated use
kubectl taint nodes -l node-pool=<pool-name> dedicated=training:NoSchedule
```

Reference: [管理Lite Cluster节点池](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0020.html)

---

## Node Management

### Node Status

| Status | Description |
|---|---|
| Running | Node is healthy and schedulable |
| NotReady | Node is unhealthy or unreachable |
| Cordoned | Node is unschedulable (maintenance mode) |
| Draining | Pods being evicted before removal |

### Key Operations

```bash
# View node details
kubectl describe node <node-name>

# Check NPU resources on a node
kubectl describe node <node-name> | grep -A 10 "Allocated resources"

# Cordon a node (prevent new pod scheduling)
kubectl cordon <node-name>

# Drain a node (evict all pods gracefully)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# Uncordon a node (re-enable scheduling)
kubectl uncordon <node-name>
```

Reference: [管理Lite Cluster节点](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0021.html)

---

## Elastic Scaling

### Scale Up (扩容)

Add nodes to the resource pool to increase compute capacity:

1. Navigate to ModelArts Console → Lite Cluster → Resource Pool
2. Click "Scale Up"
3. Specify the number of nodes to add
4. New nodes must match the existing hardware specification
5. Wait for provisioning, driver installation, and network configuration

### Scale Down (缩容)

Remove nodes from the resource pool:

1. Ensure no running jobs on the target nodes
2. Drain the nodes: `kubectl drain <node-name> --ignore-daemonsets`
3. Navigate to console and select nodes to remove
4. Confirm scale-down operation

### Precautions

- Scale-up nodes are automatically configured with the same driver and plugin versions
- Scale-down will fail if pods cannot be evicted (e.g., PodDisruptionBudget)
- Minimum 1 node must remain in the resource pool

Reference: [扩缩容Lite Cluster资源池](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0022.html)

---

## Driver Upgrades

### Pool-Wide Driver Upgrade

Upgrade the NPU driver across all nodes in the resource pool:

1. Stop all running training/inference jobs
2. Navigate to ModelArts Console → Lite Cluster → Resource Pool
3. Click "Upgrade Driver"
4. Select target driver version (check compatibility matrix first)
5. Confirm — all nodes will restart in a rolling fashion
6. Verify after upgrade:

```bash
# Check driver version on each node
kubectl get nodes -o wide
# SSH or exec into a pod to verify
npu-smi info
```

**Warning**: Pool-wide upgrade restarts all nodes. Ensure all jobs are stopped and data is saved.

Reference: [升级Lite Cluster资源池驱动](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0023.html)

### Single-Node Driver Upgrade

Upgrade the driver on a specific node without affecting the entire pool:

1. Cordon the target node: `kubectl cordon <node-name>`
2. Drain running pods: `kubectl drain <node-name> --ignore-daemonsets`
3. Navigate to console → select the specific node
4. Click "Upgrade Driver" on the individual node
5. Wait for node restart and driver installation
6. Uncordon the node: `kubectl uncordon <node-name>`

This approach is useful for:
- Testing a new driver version on a single node before pool-wide rollout
- Fixing a driver issue on one specific node
- Staged rolling upgrades with manual control

Reference: [升级Lite Cluster资源池单个节点驱动](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0028.html)

---

## Reference Documentation

- [Lite Cluster资源管理](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0017.html)
- [Lite Cluster资源管理介绍](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0018.html)
- [管理Lite Cluster资源池](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0019.html)
- [管理Lite Cluster节点池](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0020.html)
- [管理Lite Cluster节点](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0021.html)
- [扩缩容Lite Cluster资源池](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0022.html)
- [升级Lite Cluster资源池驱动](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0023.html)
- [升级Lite Cluster资源池单个节点驱动](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0028.html)
