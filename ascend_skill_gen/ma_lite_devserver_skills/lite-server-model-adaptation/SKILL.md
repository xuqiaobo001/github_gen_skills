---
name: lite-server-model-adaptation
description: Guide for adapting LLM/AIGC models for NPU training and inference, and GPT-2 for GPU training and inference on ModelArts Lite Server. This skill should be used when users need to run large language models on Ascend NPU or GPU-based Lite Server instances.
license: Apache 2.0
---

# Lite Server Model Adaptation Guide

## Overview

This skill provides guidance for adapting and running AI models on ModelArts Lite Server, covering LLM/AIGC model adaptation for Ascend NPU and GPT-2 model adaptation for GPU hardware.

---

## When to Use

- Adapting LLM or AIGC models to run on Ascend NPU via Lite Server
- Running GPT-2 training or inference on GPU-based Lite Server
- Migrating model code from GPU (CUDA) to NPU (Ascend)
- Troubleshooting model compatibility issues on Lite Server

---

## LLM/AIGC Model Adaptation for NPU

### Adaptation Workflow

```
1. Environment Preparation
   ├── Verify NPU driver and CANN version
   └── Install torch_npu and dependencies

2. Code Migration
   ├── Replace CUDA API calls with NPU equivalents
   ├── Replace backend "nccl" with "hccl"
   └── Adjust data types if needed (fp16/bf16 support)

3. Model Training
   ├── Single-node multi-card training
   └── Multi-node distributed training (HCCL)

4. Model Inference
   ├── Load trained checkpoint
   └── Run inference on NPU device
```

### Key Code Changes (CUDA → NPU)

| CUDA Code | NPU Equivalent |
|---|---|
| `import torch.cuda` | `import torch_npu` |
| `model.cuda()` | `model.npu()` |
| `tensor.cuda()` | `tensor.npu()` |
| `torch.cuda.is_available()` | `torch.npu.is_available()` |
| `torch.cuda.device_count()` | `torch.npu.device_count()` |
| `backend="nccl"` | `backend="hccl"` |
| `CUDA_VISIBLE_DEVICES` | `ASCEND_VISIBLE_DEVICES` |

### Example: LLM Training on NPU

```python
import torch
import torch_npu

# Set NPU device
torch.npu.set_device(0)

# Load model to NPU
model = AutoModelForCausalLM.from_pretrained("model_path")
model = model.npu()

# Training loop
for batch in dataloader:
    inputs = {k: v.npu() for k, v in batch.items()}
    outputs = model(**inputs)
    loss = outputs.loss
    loss.backward()
    optimizer.step()
    optimizer.zero_grad()
```

Reference: [LLM/AIGC等模型基于Lite Server适配NPU的训练推理指导](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0014.html)

---

## GPT-2 Model Adaptation for GPU

### GPU Environment Setup

```bash
# Verify GPU availability
nvidia-smi

# Install PyTorch with CUDA support
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu118
```

### GPT-2 Training Example

```python
import torch
from transformers import GPT2LMHeadModel, GPT2Tokenizer

device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

model = GPT2LMHeadModel.from_pretrained("gpt2")
model = model.to(device)

tokenizer = GPT2Tokenizer.from_pretrained("gpt2")

# Fine-tuning loop
for batch in dataloader:
    inputs = tokenizer(batch, return_tensors="pt", padding=True, truncation=True)
    inputs = {k: v.to(device) for k, v in inputs.items()}
    outputs = model(**inputs, labels=inputs["input_ids"])
    loss = outputs.loss
    loss.backward()
    optimizer.step()
    optimizer.zero_grad()
```

### GPU Inference

```python
model.eval()
with torch.no_grad():
    input_ids = tokenizer.encode("Hello, world", return_tensors="pt").to(device)
    output = model.generate(input_ids, max_length=100)
    print(tokenizer.decode(output[0], skip_special_tokens=True))
```

Reference: [GPT-2基于Lite Server适配GPU的训练推理指导](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0015.html)

---

## Troubleshooting

| Issue | Platform | Solution |
|---|---|---|
| `torch_npu` import error | NPU | Check CANN and torch_npu version compatibility |
| NPU OOM | NPU | Reduce batch size, enable gradient checkpointing |
| CUDA OOM | GPU | Reduce batch size, use mixed precision (fp16) |
| Training hang on NPU | NPU | Check HCCL config, increase timeout |
| Model output NaN | Both | Lower learning rate, check data preprocessing |

---

## Reference Documentation

- [Lite Server资源使用](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0013.html)
- [LLM/AIGC等模型基于Lite Server适配NPU的训练推理指导](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0014.html)
- [GPT-2基于Lite Server适配GPU的训练推理指导](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0015.html)
