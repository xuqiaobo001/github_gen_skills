---
name: ascend-image-builder
description: Guide for building custom Docker images for Huawei Ascend NPU on ModelArts Standard. This skill should be used when users need to create custom training or inference images with CANN toolkit, configure Ascend driver/firmware dependencies, or troubleshoot image compatibility issues on NPU hardware.
license: Apache 2.0
---

# Ascend Custom Image Builder Guide

## Overview

This skill provides guidance for building custom Docker images that run on Huawei Ascend NPU hardware within ModelArts Standard. It covers CANN toolkit installation, driver/firmware version matching, Dockerfile best practices, and image validation for both training and inference workloads.

---

## When to Use

- Building custom training images for Ascend NPU (PyTorch + Ascend)
- Building custom inference images for Ascend NPU
- Selecting correct CANN / driver / firmware version combinations
- Troubleshooting image build failures or runtime compatibility issues
- Extending ModelArts preset images with custom dependencies

---

## Version Compatibility Matrix

### Critical Rule: Version Matching

The most common failure in Ascend image building is version mismatch. The following components MUST be compatible:

```
NPU Hardware (910A/910B/910C)
  └── Driver version
       └── Firmware version
            └── CANN version
                 └── PyTorch version + torch_npu version
```

### How to Check Current Versions

```bash
# Check NPU driver version
cat /usr/local/Ascend/driver/version.info

# Check CANN version
cat /usr/local/Ascend/ascend-toolkit/latest/version.cfg

# Check torch_npu version
python -c "import torch_npu; print(torch_npu.__version__)"

# Check NPU hardware info
npu-smi info
```

Reference: [不同机型对应的软件配套版本](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/resmgmt-modelarts_0085.html)

---

## Building Training Images

### Strategy 1: Extend Preset Image (Recommended)

Start from a ModelArts preset image that already includes CANN and torch_npu:

```dockerfile
# Use ModelArts preset image as base
FROM swr.{region}.myhuaweicloud.com/atelier/pytorch_2_1_ascend:pytorch_2.1.0-cann_8.0.rc1-py_3.9-hce_2.0.2312-aarch64-snt9b-20240727152329-0f2b29b

# Install additional dependencies
RUN pip install --no-cache-dir \
    transformers==4.38.0 \
    accelerate==0.27.0 \
    deepspeed==0.14.0

# Copy training code
COPY train/ /home/ma-user/train/

# Set working directory
WORKDIR /home/ma-user/train
```

Reference: [使用预置镜像制作自定义镜像用于训练模型](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/docker-modelarts_0118.html)

### Strategy 2: Build from Scratch

When preset images do not meet requirements:

```dockerfile
# Base image (ARM architecture for Ascend)
FROM ubuntu:22.04

# Set environment variables
ENV ASCEND_HOME=/usr/local/Ascend
ENV LD_LIBRARY_PATH=${ASCEND_HOME}/ascend-toolkit/latest/lib64:$LD_LIBRARY_PATH
ENV PATH=${ASCEND_HOME}/ascend-toolkit/latest/bin:$PATH
ENV PYTHONPATH=${ASCEND_HOME}/ascend-toolkit/latest/python/site-packages:$PYTHONPATH

# Install system dependencies
RUN apt-get update && apt-get install -y \
    python3.9 python3-pip \
    gcc g++ make cmake \
    libsqlite3-dev libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# Install CANN toolkit (download from Ascend community)
COPY Ascend-cann-toolkit_8.0.RC1_linux-aarch64.run /tmp/
RUN chmod +x /tmp/Ascend-cann-toolkit_8.0.RC1_linux-aarch64.run && \
    /tmp/Ascend-cann-toolkit_8.0.RC1_linux-aarch64.run --install --quiet && \
    rm /tmp/Ascend-cann-toolkit_8.0.RC1_linux-aarch64.run

# Install PyTorch and torch_npu
RUN pip install --no-cache-dir \
    torch==2.1.0 \
    torch_npu==2.1.0.post6 \
    torchvision==0.16.0

# Create non-root user (required by ModelArts)
RUN useradd -m -u 1000 ma-user
USER ma-user
WORKDIR /home/ma-user
```

Reference: [从0制作自定义镜像用于创建训练作业（Pytorch+Ascend）](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/docker-modelarts_6031.html)

---

## Building Inference Images

### Inference Image Requirements

```dockerfile
FROM swr.{region}.myhuaweicloud.com/atelier/pytorch_2_1_ascend:latest

# Install inference framework
RUN pip install --no-cache-dir \
    flask==3.0.0 \
    gunicorn==21.2.0

# Copy model and inference code
COPY model/ /home/ma-user/model/
COPY inference.py /home/ma-user/

# ModelArts inference requires specific port and health check
EXPOSE 8080

# Entry point for inference service
ENTRYPOINT ["python", "/home/ma-user/inference.py"]
```

Key requirements for ModelArts inference images:
- Must listen on port **8080** (or configured port)
- Must implement health check endpoint at `/health`
- Must implement prediction endpoint at `/predict` or custom path
- Must run as non-root user `ma-user` (UID 1000)

Reference: [制作自定义镜像用于推理](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_23_0218.html)

---

## Image Validation

### Pre-push Validation Checklist

1. **Architecture check**: Image must be `linux/arm64` (aarch64) for Ascend hardware
2. **User check**: Default user must be `ma-user` with UID 1000
3. **CANN check**: Verify CANN toolkit is properly installed
4. **torch_npu check**: Verify torch_npu imports without error
5. **NPU device check**: Verify `torch.npu.is_available()` returns True (on NPU node)

### Validation Script

```bash
# Build the image
docker build -t my-ascend-image:latest .

# Test basic functionality
docker run --rm my-ascend-image:latest python3 -c "
import torch
import torch_npu
print(f'PyTorch: {torch.__version__}')
print(f'torch_npu: {torch_npu.__version__}')
print(f'NPU available: {torch.npu.is_available()}')
"
```

### Push to SWR (Software Repository for Container)

```bash
# Login to SWR
docker login -u {region}@{ak} -p {login_key} swr.{region}.myhuaweicloud.com

# Tag and push
docker tag my-ascend-image:latest \
  swr.{region}.myhuaweicloud.com/{organization}/my-ascend-image:latest
docker push swr.{region}.myhuaweicloud.com/{organization}/my-ascend-image:latest
```

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| `libascendcl.so not found` | CANN not installed or LD_LIBRARY_PATH wrong | Verify CANN install path and ENV settings |
| `torch_npu` import error | Version mismatch with PyTorch | Ensure torch_npu version matches PyTorch version |
| `Permission denied` at runtime | Image not using ma-user | Add `USER ma-user` in Dockerfile |
| Image too large (>30GB) | Too many layers or unnecessary files | Use multi-stage build, clean apt/pip cache |
| `exec format error` | Wrong architecture (x86 vs arm64) | Build on aarch64 or use `--platform linux/arm64` |

---

## Reference Documentation

- [自定义镜像使用场景](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_23_0084.html)
- [ModelArts支持的预置镜像列表](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/docker-modelarts_6014.html)
- [训练作业的自定义镜像制作流程](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/develop-modelarts-10079.html)
- [模型的自定义镜像制作流程](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_23_0219.html)
