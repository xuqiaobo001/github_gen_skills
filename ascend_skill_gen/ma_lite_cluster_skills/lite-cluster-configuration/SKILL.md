---
name: lite-cluster-configuration
description: Guide for configuring ModelArts Lite Cluster resources, including network setup, kubectl tool configuration, storage configuration, driver configuration, and image pre-warming. This skill should be used when users need to set up networking, storage, kubectl access, or optimize image deployment on Lite Cluster.
license: Apache 2.0
---

# Lite Cluster Configuration Guide

## Overview

This skill provides guidance for configuring ModelArts Lite Cluster resources after provisioning. It covers the configuration flow, network setup, kubectl access, storage, driver configuration, and image pre-warming for optimized container deployment.

---

## When to Use

- Setting up network connectivity for a new Lite Cluster
- Configuring kubectl to manage cluster resources
- Configuring shared storage (SFS Turbo, OBS) for training data
- Configuring or updating NPU drivers on cluster nodes
- Setting up image pre-warming to accelerate pod startup

---

## Configuration Flow

```
Resource Provisioning Complete
  │
  ├── 1. Configure Network (Required)
  │     └── VPC, subnet, security groups, EIP
  │
  ├── 2. Configure kubectl (Required)
  │     └── Download kubeconfig, install kubectl
  │
  ├── 3. Configure Storage (Required)
  │     └── SFS Turbo / OBS for training data
  │
  ├── 4. Configure Driver (Optional)
  │     └── Update NPU driver if needed
  │
  └── 5. Configure Image Pre-warming (Optional)
        └── Pre-pull large container images to nodes
```

Reference: [Lite Cluster资源配置流程](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0007.html)

---

## Network Configuration

### Network Architecture

Lite Cluster uses multiple network planes for different purposes:

| Network Plane | Purpose | Protocol |
|---|---|---|
| Management plane | kubectl access, node management | TCP/HTTPS |
| Data plane | Training data transfer, storage access | TCP |
| Compute plane | Inter-node NPU communication (HCCS/RoCE) | RDMA/RoCE v2 |

### Configuration Steps

1. **VPC and Subnet**: Ensure the Lite Cluster resource pool is associated with the correct VPC and subnet
2. **Security Groups**: Configure inbound/outbound rules for:
   - SSH access (port 22) from management hosts
   - Kubernetes API (port 6443) for kubectl
   - Inter-node communication ports for distributed training
3. **EIP (Optional)**: Bind an Elastic IP for external access if needed

Reference: [配置Lite Cluster网络](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0008.html)

---

## kubectl Configuration

### Overview

kubectl is the standard Kubernetes command-line tool used to manage Lite Cluster resources, submit jobs, and inspect workloads.

### Setup Steps

1. **Install kubectl**: Install a kubectl version compatible with the cluster's Kubernetes version

```bash
# Download kubectl (example for v1.25)
curl -LO "https://dl.k8s.io/release/v1.25.0/bin/linux/amd64/kubectl"
chmod +x kubectl
mv kubectl /usr/local/bin/
```

2. **Download kubeconfig**: Obtain the kubeconfig file from ModelArts Console → Lite Cluster → Cluster Details
3. **Configure access**:

```bash
# Set kubeconfig path
export KUBECONFIG=/path/to/kubeconfig.yaml

# Verify connectivity
kubectl get nodes
kubectl get pods -A
```

### Verify Cluster Access

```bash
# Check node status
kubectl get nodes -o wide

# Check NPU resources
kubectl describe node <node-name> | grep -A 5 "Allocatable"

# Check system pods
kubectl get pods -n kube-system
```

Reference: [配置kubectl工具](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0009.html)

---

## Storage Configuration

### Supported Storage Types

| Storage Type | Use Case | Access Mode |
|---|---|---|
| SFS Turbo | Shared training data, checkpoints | ReadWriteMany (RWX) |
| OBS | Large dataset storage, model artifacts | Via s3fs or obsfs mount |
| EVS | Node-local high-performance storage | ReadWriteOnce (RWO) |

### SFS Turbo Configuration

1. Create an SFS Turbo file system in the same VPC as the Lite Cluster
2. Create a PersistentVolume (PV) and PersistentVolumeClaim (PVC) in Kubernetes:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: sfs-turbo-pv
spec:
  capacity:
    storage: 500Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: <sfs-turbo-ip>
    path: /
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: sfs-turbo-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 500Gi
```

3. Mount the PVC in training pods to access shared data

Reference: [配置Lite Cluster存储](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0010.html)

---

## Driver Configuration (Optional)

### When to Configure

- Default driver version does not meet framework requirements
- Need to upgrade driver for new CANN or torch_npu version
- Hardware-specific driver optimization required

### Configuration Steps

1. Check current driver version on cluster nodes:

```bash
kubectl exec -it <pod-name> -- npu-smi info
```

2. Navigate to ModelArts Console → Lite Cluster → Resource Pool
3. Select "Configure Driver"
4. Choose the target driver version from the compatibility matrix
5. Confirm — nodes will restart during the upgrade

**Warning**: Driver changes require node restart. Stop all running jobs before proceeding.

Reference: [（可选）配置驱动](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0011.html)

---

## Image Pre-warming (Optional)

### Why Pre-warm Images

AI training container images are often very large (10-50 GB+). Without pre-warming, pod startup is delayed by image pull time. Image pre-warming pre-pulls specified images to all nodes in advance.

### Benefits

| Without Pre-warming | With Pre-warming |
|---|---|
| Pod startup delayed by image pull | Near-instant pod startup |
| Network congestion during job submission | Images already cached on nodes |
| Timeout risk for very large images | Reliable image availability |

### Configuration Steps

1. Navigate to ModelArts Console → Lite Cluster → Image Pre-warming
2. Specify the container image address (SWR image URL)
3. Select target node pool(s)
4. Trigger pre-warming
5. Monitor pre-warming progress across nodes

### Verify Pre-warming

```bash
# Check if image exists on a specific node
kubectl debug node/<node-name> -it --image=busybox -- crictl images | grep <image-name>
```

Reference: [（可选）配置镜像预热](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0012.html)

---

## Reference Documentation

- [Lite Cluster资源配置](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0006.html)
- [Lite Cluster资源配置流程](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0007.html)
- [配置Lite Cluster网络](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0008.html)
- [配置kubectl工具](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0009.html)
- [配置Lite Cluster存储](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0010.html)
- [（可选）配置驱动](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0011.html)
- [（可选）配置镜像预热](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0012.html)
