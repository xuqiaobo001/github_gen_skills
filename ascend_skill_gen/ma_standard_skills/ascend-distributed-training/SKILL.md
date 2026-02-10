---
name: ascend-distributed-training
description: Guide for configuring and running distributed model training on Huawei Ascend NPU clusters. This skill should be used when users need to set up single-node multi-card or multi-node multi-card distributed training using PyTorch DDP on Ascend NPU, configure HCCL communication, or troubleshoot distributed training issues.
license: Apache 2.0
---

# Ascend Distributed Training Guide

## Overview

This skill provides guidance for setting up and running distributed model training on Huawei Ascend NPU hardware using ModelArts Standard. It covers single-node multi-card (DataParallel), multi-node multi-card (DistributedDataParallel) configurations, HCCL communication setup, and common troubleshooting patterns.

---

## When to Use

- Setting up single-node multi-card training on Ascend NPU
- Configuring multi-node multi-card DDP training on Ascend NPU
- Configuring HCCL (Huawei Collective Communication Library) parameters
- Writing distributed training launch scripts for ModelArts
- Troubleshooting distributed training hang or crash issues on NPU

---

## Distributed Training Architectures

### Architecture 1: Single-Node Multi-Card (DataParallel)

Suitable for models that fit in a single node with multiple NPU cards.

```python
import torch
import torch_npu

model = MyModel().npu()
# Use DataParallel across all available NPU devices
model = torch.nn.DataParallel(model, device_ids=[0, 1, 2, 3, 4, 5, 6, 7])
```

**Limitations**: DataParallel uses a single process with GIL contention. Prefer DDP for better performance.

### Architecture 2: Multi-Node Multi-Card (DistributedDataParallel)

Recommended approach for large-scale training across multiple nodes.

```python
import os
import torch
import torch.distributed as dist
import torch_npu

def init_distributed():
    rank = int(os.environ["RANK"])
    local_rank = int(os.environ["LOCAL_RANK"])
    world_size = int(os.environ["WORLD_SIZE"])

    # Use HCCL backend for Ascend NPU
    dist.init_process_group(
        backend="hccl",
        rank=rank,
        world_size=world_size
    )
    torch.npu.set_device(local_rank)
    return rank, local_rank, world_size

def main():
    rank, local_rank, world_size = init_distributed()

    model = MyModel().npu()
    model = torch.nn.parallel.DistributedDataParallel(
        model,
        device_ids=[local_rank],
        output_device=local_rank
    )

    # Create distributed sampler
    train_sampler = torch.utils.data.distributed.DistributedSampler(
        train_dataset,
        num_replicas=world_size,
        rank=rank
    )

    train_loader = torch.utils.data.DataLoader(
        train_dataset,
        batch_size=batch_size_per_card,
        sampler=train_sampler,
        num_workers=4,
        pin_memory=True
    )

    for epoch in range(num_epochs):
        train_sampler.set_epoch(epoch)
        train_one_epoch(model, train_loader, optimizer)

    dist.destroy_process_group()
```

---

## HCCL Communication Configuration

### Key Environment Variables

```bash
# HCCL timeout settings (seconds)
export HCCL_CONNECT_TIMEOUT=1200
export HCCL_EXEC_TIMEOUT=1800

# HCCL whitelist (disable for cross-node communication)
export HCCL_WHITELIST_DISABLE=1

# HCCL network interface
export HCCL_IF_BASE_PORT=29500

# Enable combined communication optimization
export COMBINED_ENABLE=1

# HCCL buffer size (bytes, default 200MB)
export HCCL_BUFFSIZE=209715200
```

### HCCL vs NCCL Key Differences

| Feature | NCCL (GPU) | HCCL (NPU) |
|---|---|---|
| Backend name | `"nccl"` | `"hccl"` |
| Device type | `cuda` | `npu` |
| Communication link | NVLink / PCIe / InfiniBand | HCCS / RoCE |
| Config file | Not required | Optional `hccl.json` for complex topologies |
| Default port | 29500 | 29500 |

---

## ModelArts Training Job Configuration

### Launch Script for ModelArts

When submitting a distributed training job on ModelArts Standard, the platform automatically sets environment variables:

```bash
# Auto-set by ModelArts
RANK          # Global rank of current process
LOCAL_RANK    # Local rank within the node
WORLD_SIZE    # Total number of processes
MASTER_ADDR   # Address of rank-0 node
MASTER_PORT   # Port of rank-0 node
```

### Training Job Submission Parameters

When creating a training job on ModelArts:

- **Compute nodes**: Set to the number of nodes (e.g., 2 for 2-node training)
- **Resource specification**: Select Ascend NPU flavor (e.g., `modelarts.pool.visual.8xAscend910`)
- **Boot command**: `python -m torch.distributed.launch --nproc_per_node=8 train.py`
- Or use `torchrun`: `torchrun --nproc_per_node=8 train.py`

Reference: [创建多机多卡的分布式训练（DistributedDataParallel）](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts-distributed-0008.html)

---

## Performance Optimization

### Gradient Communication Optimization

```python
# Enable gradient bucketing (default in DDP, tune bucket size)
model = torch.nn.parallel.DistributedDataParallel(
    model,
    device_ids=[local_rank],
    bucket_cap_mb=25  # Tune based on model size
)
```

### Data Loading Optimization

```python
# Use persistent workers and prefetch
train_loader = torch.utils.data.DataLoader(
    dataset,
    batch_size=batch_size,
    sampler=train_sampler,
    num_workers=8,
    pin_memory=True,
    persistent_workers=True,
    prefetch_factor=2
)
```

### Mixed Precision with DDP

```python
from torch.npu.amp import autocast, GradScaler

scaler = GradScaler()
for data, target in train_loader:
    optimizer.zero_grad()
    with autocast():
        output = model(data.npu())
        loss = criterion(output, target.npu())
    scaler.scale(loss).backward()
    scaler.step(optimizer)
    scaler.update()
```

---

## Troubleshooting

### Common Issues

| Issue | Symptom | Solution |
|---|---|---|
| HCCL init timeout | Process hangs at `init_process_group` | Increase `HCCL_CONNECT_TIMEOUT`, verify network connectivity between nodes |
| Rank mismatch | `RuntimeError: rank X is not in group` | Verify `RANK`, `WORLD_SIZE` env vars are correctly set |
| OOM on NPU | `NPU out of memory` during DDP | Reduce per-card batch size, enable gradient checkpointing |
| Training hang | No progress after N iterations | Enable hang detection, check HCCL logs, verify all ranks reach same barrier |
| Gradient NaN | Loss becomes NaN during training | Check learning rate scaling with world_size, verify mixed precision config |

### Diagnostic Commands

```bash
# Check NPU device status
npu-smi info

# Check HCCL connectivity
hccl_test -b 8 -e 128M -f 2 -g 8

# Monitor NPU utilization during training
watch -n 1 npu-smi info
```

---

## Reference Documentation

- [分布式训练功能介绍](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts-distributed-0001.html)
- [创建单机多卡的分布式训练（DataParallel）](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts-distributed-0007.html)
- [创建多机多卡的分布式训练（DistributedDataParallel）](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts-distributed-0008.html)
- [示例：创建DDP分布式训练（PyTorch+NPU）](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts-distributed-0012.html)
- [超节点亲和组实例数配置](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/develop-modelarts-0011.html)
