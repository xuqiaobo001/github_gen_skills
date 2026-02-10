---
name: lite-cluster-inference-and-tools
description: Guide for running inference workloads, using Ascend FaultDiag diagnostic tool, mounting SFS Turbo storage, configuring high-availability redundant nodes, and cross-region service access on Lite Cluster. This skill should be used when users need to deploy inference services, diagnose NPU faults, configure shared storage, or set up HA and cross-region access on Lite Cluster.
license: Apache 2.0
---

# Lite Cluster Inference & Tools Guide

## Overview

This skill provides guidance for inference workloads and operational tools on ModelArts Lite Cluster, covering Snt9B inference deployment, Ascend FaultDiag log diagnosis, SFS Turbo mounting, high-availability redundant nodes, and cross-region service access.

---

## When to Use

- Deploying inference services on Lite Cluster with Snt9B hardware
- Using Ascend FaultDiag tool to diagnose NPU log issues
- Mounting SFS Turbo shared file system to Lite Cluster pods
- Configuring high-availability redundant nodes for fault tolerance
- Accessing services across regions from Lite Cluster

---

## Snt9B Inference Deployment

### Overview

Lite Cluster supports deploying inference services on Snt9B hardware using Kubernetes Deployments or VolcanoJobs, enabling low-latency AI model serving with Ascend 910B NPUs.

### Inference Job Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: inference-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: inference
  template:
    metadata:
      labels:
        app: inference
    spec:
      containers:
        - name: inference
          image: <swr-inference-image-url>
          command: ["/bin/bash", "-c"]
          args: ["python serve.py --model /data/model --port 8080"]
          ports:
            - containerPort: 8080
          resources:
            limits:
              huawei.com/ascend-910b: "1"
          volumeMounts:
            - name: model-data
              mountPath: /data
      volumes:
        - name: model-data
          persistentVolumeClaim:
            claimName: sfs-turbo-pvc
```

### Expose Inference Service

```bash
# Create a ClusterIP service
kubectl expose deployment inference-service --port=8080 --target-port=8080

# Or create a NodePort service for external access
kubectl expose deployment inference-service --type=NodePort --port=8080
```

Reference: [在Lite Cluster资源池上使用Snt9B完成推理任务](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0016.html)

---

## Ascend FaultDiag Tool

### What is FaultDiag

Ascend FaultDiag is a diagnostic tool that analyzes NPU logs to identify hardware faults, communication errors, and training failures. It automates log parsing and provides structured fault reports.

### Diagnostic Capabilities

| Category | What It Detects |
|---|---|
| Hardware faults | ECC errors, NPU device lost, overtemperature |
| Communication errors | HCCS link down, RoCE timeout, HCCL failures |
| Training errors | Operator execution failures, memory overflow |
| Driver issues | Driver crash, firmware mismatch |

### Usage on Lite Cluster

1. Collect logs from the affected pod or node
2. Run FaultDiag tool against the collected logs:

```bash
# Run FaultDiag inside a pod with access to NPU logs
ascend-faultdiag --log-path /var/log/npu/ --output /tmp/diag_report/

# View diagnosis report
cat /tmp/diag_report/summary.json
```

3. Review the structured report for identified faults and recommended actions

Reference: [在Lite Cluster资源池上使用Ascend FaultDiag工具完成日志诊断](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0030.html)

---

## SFS Turbo Mounting

### Overview

SFS Turbo (Scalable File Service Turbo) provides high-performance shared file storage for Lite Cluster pods. It is commonly used for training datasets, model checkpoints, and shared configuration files.

### Mounting Steps

1. Create an SFS Turbo file system in the same VPC as the Lite Cluster
2. Create PV and PVC resources (see lite-cluster-configuration skill for YAML examples)
3. Mount in pod spec:

```yaml
volumeMounts:
  - name: sfs-data
    mountPath: /data
volumes:
  - name: sfs-data
    persistentVolumeClaim:
      claimName: sfs-turbo-pvc
```

### Performance Considerations

| Tier | Bandwidth | Use Case |
|---|---|---|
| SFS Turbo Standard | Up to 150 MB/s | Small datasets, config files |
| SFS Turbo Performance | Up to 350 MB/s | Medium datasets |
| SFS Turbo HPC | Up to 2 GB/s+ | Large-scale training data |

Reference: [在Lite Cluster挂载SFS Turbo](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0043.html)

---

## High-Availability Redundant Nodes

### Overview

High-availability (HA) redundant nodes are standby nodes in the Lite Cluster resource pool that automatically replace faulty nodes during distributed training, minimizing job interruption.

### How It Works

```
Normal Operation:
  Node-1 [Active] ── Node-2 [Active] ── Node-3 [Active] ── Node-4 [Standby]

Fault Detected:
  Node-1 [Active] ── Node-2 [FAULT] ── Node-3 [Active] ── Node-4 [Standby]

Auto Recovery:
  Node-1 [Active] ── Node-4 [Replacing] ── Node-3 [Active] ── Node-2 [Repairing]
```

### Configuration Steps

1. Navigate to ModelArts Console → Lite Cluster → Resource Pool
2. Select "HA Configuration"
3. Specify the number of redundant (standby) nodes
4. Enable automatic failover
5. Configure failover policy (immediate or after retry)

### Considerations

- Redundant nodes consume resources but do not run workloads until failover
- Recommended ratio: 1 standby node per 8-16 active nodes
- Failover triggers automatic ranktable regeneration for distributed training

Reference: [在Lite Cluster资源池设置并启用高可用冗余节点](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0044.html)

---

## Cross-Region Service Access

### Overview

Lite Cluster resources may need to access services (OBS, SWR, APIs) deployed in other Huawei Cloud regions. Cross-region access requires proper network routing and endpoint configuration.

### Common Cross-Region Scenarios

| Scenario | Source | Target |
|---|---|---|
| Pull images from remote SWR | Lite Cluster region | SWR in another region |
| Access OBS datasets | Lite Cluster region | OBS in another region |
| Call remote API services | Lite Cluster pods | Services in another region |

### Configuration Steps

1. Ensure VPC peering or Cloud Connect is established between regions
2. Configure DNS resolution for cross-region endpoints
3. Update security group rules to allow cross-region traffic
4. Configure proxy or endpoint settings in pod environment variables if needed

Reference: [在Lite Cluster跨区域访问其他服务](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0045.html)

---

## Reference Documentation

- [Lite Cluster资源使用](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0013.html)
- [在Lite Cluster资源池上使用Snt9B完成推理任务](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0016.html)
- [在Lite Cluster资源池上使用Ascend FaultDiag工具完成日志诊断](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0030.html)
- [在Lite Cluster挂载SFS Turbo](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0043.html)
- [在Lite Cluster资源池设置并启用高可用冗余节点](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0044.html)
- [在Lite Cluster跨区域访问其他服务](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0045.html)
