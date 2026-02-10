---
name: ascend-training-code-migration
description: Guide for migrating AI training code from NVIDIA GPU to Huawei Ascend NPU. This skill should be used when users need to convert PyTorch/TensorFlow GPU training scripts to run on Ascend NPU hardware, including torch_npu adaptation, device mapping, operator compatibility checks, and CANN toolkit integration.
license: Apache 2.0
---

# Ascend Training Code Migration Guide (GPU → NPU)

## Overview

This skill provides systematic guidance for migrating AI model training code from NVIDIA GPU to Huawei Ascend NPU. It covers PyTorch and TensorFlow framework adaptation, operator compatibility analysis, and performance optimization on the Ascend platform.

---

## When to Use

- Migrating existing PyTorch GPU training scripts to Ascend NPU
- Migrating TensorFlow GPU training scripts to Ascend NPU
- Checking operator compatibility between CUDA and CANN
- Resolving NPU-specific runtime errors during training
- Optimizing training performance after migration to NPU

---

## Migration Process

### Phase 1: Environment Assessment

#### 1.1 Identify Source Framework and Version

Before migration, determine the exact framework configuration:

- **PyTorch version**: Check `torch.__version__` and CUDA version
- **TensorFlow version**: Check `tf.__version__` and GPU support
- **Third-party dependencies**: List all packages that use CUDA directly (e.g., apex, deepspeed, flash-attention)

#### 1.2 Identify Target Ascend Environment

Confirm the target Ascend environment:

- **CANN version**: The Ascend heterogeneous computing architecture (e.g., CANN 7.0, 8.0)
- **NPU model**: Ascend 910A, 910B, 910C, or other variants
- **torch_npu version**: Must match the PyTorch and CANN versions
- **Driver/Firmware version**: Must be compatible with CANN version

Reference: [不同机型对应的软件配套版本](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0085.html)

---

### Phase 2: Core Code Migration (PyTorch)

#### 2.1 Import Adaptation

Replace or add NPU-related imports:

```python
# Before (GPU)
import torch
import torch.cuda

# After (NPU)
import torch
import torch_npu  # Must import after torch
```

#### 2.2 Device Replacement

Systematically replace all CUDA device references:

```python
# Before (GPU)
device = torch.device("cuda:0")
model = model.cuda()
tensor = tensor.to("cuda")
torch.cuda.is_available()
torch.cuda.device_count()
torch.cuda.set_device(local_rank)
torch.cuda.synchronize()

# After (NPU)
device = torch.device("npu:0")
model = model.npu()
tensor = tensor.to("npu")
torch.npu.is_available()
torch.npu.device_count()
torch.npu.set_device(local_rank)
torch.npu.synchronize()
```

#### 2.3 Distributed Training Backend Replacement

```python
# Before (GPU) - NCCL backend
torch.distributed.init_process_group(backend="nccl")

# After (NPU) - HCCL backend
torch.distributed.init_process_group(backend="hccl")
```

#### 2.4 Mixed Precision Adaptation

```python
# Before (GPU) - using torch.cuda.amp
from torch.cuda.amp import autocast, GradScaler
scaler = GradScaler()
with autocast():
    output = model(input)

# After (NPU) - using torch.npu.amp or torch.amp
from torch.npu.amp import autocast, GradScaler
scaler = GradScaler()
with autocast():
    output = model(input)
```

#### 2.5 APEX Adaptation (if used)

```python
# Before (GPU)
from apex import amp
model, optimizer = amp.initialize(model, optimizer, opt_level="O1")

# After (NPU) - replace with native AMP or use Ascend-adapted apex
# Option 1: Native PyTorch AMP (recommended)
scaler = torch.npu.amp.GradScaler()

# Option 2: Ascend-adapted apex (if needed)
from apex import amp
# Ensure using Ascend-compatible apex version
```

---

### Phase 3: Operator Compatibility Check

#### 3.1 Common Incompatible Operators

The following operators may require special handling on Ascend NPU:

| GPU Operator | NPU Status | Solution |
|---|---|---|
| `torch.cuda.amp` | Replaced | Use `torch.npu.amp` |
| `flash_attention` (CUDA) | Not compatible | Use Ascend FlashAttention or `torch_npu.npu_fusion_attention` |
| `apex.normalization.FusedLayerNorm` | Replaced | Use `torch_npu` fused operators |
| Custom CUDA kernels (.cu files) | Not compatible | Rewrite using Ascend C or use equivalent CANN operators |
| `torch.backends.cudnn` | Not applicable | Remove or replace with NPU equivalents |

#### 3.2 Operator Fallback Strategy

When an operator is not supported on NPU:

1. Check if `torch_npu` provides an equivalent fused operator
2. Check if CANN provides a compatible operator via `aclnn` interface
3. Fall back to CPU computation for unsupported operators (performance impact)
4. Rewrite using Ascend C for performance-critical custom operators

---

### Phase 4: Environment Variable Configuration

#### 4.1 Key NPU Environment Variables

```bash
# Set visible NPU devices (equivalent to CUDA_VISIBLE_DEVICES)
export ASCEND_VISIBLE_DEVICES=0,1,2,3

# HCCL configuration for distributed training
export HCCL_CONNECT_TIMEOUT=1200
export HCCL_EXEC_TIMEOUT=1200

# Performance tuning
export COMBINED_ENABLE=1
export HCCL_WHITELIST_DISABLE=1

# Logging and debugging
export ASCEND_GLOBAL_LOG_LEVEL=3  # 0=DEBUG, 1=INFO, 2=WARNING, 3=ERROR
export ASCEND_SLOG_PRINT_TO_STDOUT=0
```

---

### Phase 5: Validation and Testing

#### 5.1 Functional Validation

1. Run a single-step forward pass to verify model loads correctly on NPU
2. Run a single-step backward pass to verify gradient computation
3. Compare output tensors between GPU and NPU (allow small numerical differences due to precision)
4. Run a few training iterations and verify loss convergence trend

#### 5.2 Common Migration Issues

| Issue | Symptom | Solution |
|---|---|---|
| Operator not supported | `RuntimeError: not implemented for 'PrivateUse1'` | Check operator compatibility, use fallback |
| Data type mismatch | `RuntimeError: expected scalar type Float but found Half` | Explicitly cast tensors to correct dtype |
| HCCL timeout | `HCCL timeout` during distributed init | Increase `HCCL_CONNECT_TIMEOUT`, check network |
| Memory overflow | `NPU out of memory` | Reduce batch size, enable gradient checkpointing |
| Precision difference | Loss diverges compared to GPU | Check mixed precision config, use FP32 for sensitive ops |

---

## Reference Documentation

- [开发用于预置框架训练的代码](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/develop-modelarts-0008.html)
- [示例：创建DDP分布式训练（PyTorch+NPU）](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts-distributed-0012.html)
- [从0制作自定义镜像用于创建训练作业（Pytorch+Ascend）](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/docker-modelarts_6031.html)
- [不同机型对应的软件配套版本](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0085.html)
