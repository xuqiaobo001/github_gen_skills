---
name: lite-cluster-distributed-training
description: Guide for running distributed training on ModelArts Lite Cluster, including Snt9B-based distributed training and ranktable-based PyTorch NPU distributed training. This skill should be used when users need to submit and manage distributed training jobs on Lite Cluster using Volcano scheduler and Ascend NPU resources.
license: Apache 2.0
---

# Lite Cluster Distributed Training Guide

## Overview

This skill provides guidance for running distributed training workloads on ModelArts Lite Cluster. It covers Snt9B-based distributed training and ranktable routing-based PyTorch NPU distributed training using the Volcano scheduler.

---

## When to Use

- Submitting distributed training jobs on Lite Cluster with Snt9B hardware
- Configuring ranktable-based PyTorch NPU distributed training
- Understanding Volcano job scheduling for multi-node training
- Troubleshooting distributed training failures on Lite Cluster

---

## Distributed Training Architecture

### Key Components

| Component | Role |
|---|---|
| Volcano scheduler | AI-optimized K8s job scheduler for gang scheduling |
| VolcanoJob | Custom resource for defining distributed training jobs |
| Ranktable | Routing configuration mapping ranks to NPU devices |
| HCCL | Huawei Collective Communication Library for inter-NPU communication |
| Ring Controller | Manages NPU communication topology |

### Scheduling Model

```
User submits VolcanoJob
  │
  ├── Volcano scheduler performs gang scheduling
  │     └── All worker pods scheduled simultaneously (or none)
  │
  ├── Ring Controller generates ranktable
  │     └── Maps rank IDs to node IPs and NPU device IDs
  │
  └── Worker pods start training
        └── HCCL uses ranktable for collective communication
```

---

## Snt9B Distributed Training

### What is Snt9B

Snt9B is a high-performance Ascend NPU server specification commonly used in Lite Cluster for large-scale distributed training. Each Snt9B node typically contains 8 Ascend 910B NPUs interconnected via HCCS.

### Job Submission

Submit a distributed training job using a VolcanoJob YAML:

```yaml
apiVersion: batch.volcano.sh/v1alpha1
kind: Job
metadata:
  name: distributed-training-job
  labels:
    ring-controller.atlas: ascend-910b
spec:
  minAvailable: 2
  schedulerName: volcano
  plugins:
    env: []
    svc: []
  tasks:
    - replicas: 2
      name: worker
      policies:
        - event: TaskCompleted
          action: CompleteJob
      template:
        spec:
          containers:
            - name: worker
              image: <swr-image-url>
              command: ["/bin/bash", "-c"]
              args: ["python train.py"]
              resources:
                limits:
                  huawei.com/ascend-910b: "8"
              volumeMounts:
                - name: data
                  mountPath: /data
          volumes:
            - name: data
              persistentVolumeClaim:
                claimName: sfs-turbo-pvc
          restartPolicy: OnFailure
```

### Key Configuration Parameters

| Parameter | Description | Example |
|---|---|---|
| `minAvailable` | Minimum pods for gang scheduling | 2 (for 2-node training) |
| `replicas` | Number of worker pods | 2 |
| `huawei.com/ascend-910b` | NPU resource request per pod | 8 (all NPUs on node) |
| `schedulerName` | Must be `volcano` | volcano |
| `ring-controller.atlas` | NPU type label for ring controller | ascend-910b |

### Submit and Monitor

```bash
# Submit job
kubectl apply -f training-job.yaml

# Check job status
kubectl get vcjob

# Check pod status
kubectl get pods -l volcano.sh/job-name=distributed-training-job

# View training logs
kubectl logs <pod-name> -f
```

Reference: [在Lite Cluster资源池上使用Snt9B完成分布式训练任务](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0014.html)

---

## Ranktable-Based PyTorch NPU Distributed Training

### What is Ranktable Routing

Ranktable is a JSON configuration file that defines the mapping between training ranks and physical NPU devices across nodes. It enables PyTorch NPU distributed training by providing explicit routing information for HCCL collective communication.

### Ranktable Structure

```json
{
  "status": "completed",
  "group_count": "1",
  "group_list": [
    {
      "group_name": "group0",
      "device_count": "16",
      "instance_count": "2",
      "instance_list": [
        {
          "instance_name": "node-0",
          "devices": [
            {"device_id": "0", "device_ip": "192.168.1.1"},
            {"device_id": "1", "device_ip": "192.168.1.2"}
          ]
        }
      ]
    }
  ]
}
```

### Key Environment Variables

| Variable | Description | Example |
|---|---|---|
| `RANK_TABLE_FILE` | Path to ranktable JSON | `/user/config/hccl.json` |
| `RANK_SIZE` | Total number of NPU devices | 16 |
| `RANK_ID` | Current process rank ID | 0 |
| `DEVICE_ID` | Local NPU device ID | 0 |

### Training Script Integration

```python
import torch
import torch_npu
from torch_npu.contrib import transfer_to_npu

# Initialize distributed process group
torch.distributed.init_process_group(backend='hccl')

# Training code uses standard PyTorch DDP
model = torch.nn.parallel.DistributedDataParallel(model)
```

Reference: [在Lite Cluster资源池上使用ranktable路由规划完成PyTorch NPU分布式训练](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0015.html)

---

## Troubleshooting

| Issue | Possible Cause | Solution |
|---|---|---|
| Pods stuck in Pending | Insufficient NPU resources | Check node allocatable resources with `kubectl describe node` |
| Gang scheduling timeout | Not enough nodes available | Verify minAvailable matches available node count |
| HCCL initialization failure | Ranktable misconfiguration | Check RANK_TABLE_FILE path and content |
| Training hangs | Network communication issue | Check RoCE network and HCCS link status |
| OOM on NPU | HBM exhausted | Reduce batch size or enable gradient checkpointing |

---

## Reference Documentation

- [Lite Cluster资源使用](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0013.html)
- [在Lite Cluster资源池上使用Snt9B完成分布式训练任务](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0014.html)
- [在Lite Cluster资源池上使用ranktable路由规划完成PyTorch NPU分布式训练](https://support.huaweicloud.com/intl/zh-cn/usermanual-cluster-modelarts/umn-cluster-modelarts-0015.html)
