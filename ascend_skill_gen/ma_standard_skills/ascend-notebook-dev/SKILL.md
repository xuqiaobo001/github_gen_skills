---
name: ascend-notebook-dev
description: Guide for setting up and using Notebook development environments on Huawei Ascend NPU via ModelArts Standard. This skill should be used when users need to create NPU-powered Notebook instances, connect via JupyterLab/PyCharm/VS Code, use ma-cli commands, or perform data operations with MoXing on Ascend hardware.
license: Apache 2.0
---

# Ascend Notebook Development Environment Guide

## Overview

This skill provides guidance for creating and using Notebook development environments on Huawei Ascend NPU within ModelArts Standard. It covers instance creation, remote IDE connections, ma-cli command usage, and MoXing data operations.

---

## When to Use

- Creating a Notebook instance with Ascend NPU resources
- Connecting to Notebook via JupyterLab, PyCharm, or VS Code
- Using ma-cli commands for image building and job management
- Performing OBS data operations with MoXing framework
- Debugging and prototyping AI models on NPU interactively

---

## Creating a Notebook Instance

### Key Configuration Parameters

| Parameter | Description | Recommendation |
|---|---|---|
| Instance name | Unique identifier | Descriptive name with project prefix |
| Resource flavor | NPU specification | Select Ascend 910 flavor for training dev |
| Image | Base software environment | Use preset image with matching CANN version |
| Storage | Persistent storage type | EVS for code, OBS for large datasets |
| Auto-stop | Idle timeout | Enable to avoid resource waste |

### Verify NPU Access in Notebook

After creating and starting the instance, verify NPU is accessible:

```python
import torch
import torch_npu

print(f"NPU available: {torch.npu.is_available()}")
print(f"NPU count: {torch.npu.device_count()}")
print(f"NPU name: {torch.npu.get_device_name(0)}")
```

Reference: [创建Notebook实例（新版页面）](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/devtool-modelarts_3495.html)

---

## Remote IDE Connections

### Option 1: JupyterLab (Browser-based)

The default web-based development environment:

- Open directly from ModelArts console
- Supports `.ipynb` notebooks and terminal access
- Built-in Git integration for cloning repositories
- Supports MindInsight and TensorBoard visualization

Reference: [使用JupyterLab在线开发和调试代码](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/devtool-modelarts_0012.html)

### Option 2: VS Code Remote

Connect VS Code to Notebook via SSH:

1. Install "Remote - SSH" extension in VS Code
2. Get SSH connection info from Notebook instance details
3. Configure SSH config file with host, port, and key
4. Connect and open remote workspace

Alternatively, use VS Code ToolKit plugin for one-click connection.

Reference: [VS Code手动连接Notebook](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/devtool-modelarts_0208.html)

### Option 3: PyCharm Remote

Connect PyCharm Professional to Notebook:

1. Use PyCharm Toolkit plugin (recommended) for automatic setup
2. Or configure SSH interpreter manually

Reference: [使用PyCharm Toolkit插件连接Notebook](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/devtool-modelarts_0016.html)

---

## ma-cli Command Reference

ModelArts CLI (ma-cli) is a command-line tool available within Notebook for common operations.

### Authentication

```bash
# Configure AK/SK credentials
ma-cli configure

# Verify configuration
ma-cli configure show
```

### Image Building

```bash
# Build custom image from Dockerfile
ma-cli image build -t my-image:latest -f Dockerfile .

# List available images
ma-cli image list

# Push image to SWR
ma-cli image push my-image:latest
```

### Training Job Management

```bash
# Submit a training job
ma-cli ma-job submit --name my-job \
  --image swr.{region}.myhuaweicloud.com/{org}/my-image:latest \
  --command "python train.py" \
  --pool-id {pool_id} \
  --node-count 1

# List training jobs
ma-cli ma-job list

# View job logs
ma-cli ma-job log --job-id {job_id}
```

### OBS Data Copy

```bash
# Copy local file to OBS
ma-cli obs-copy /local/path/ obs://bucket/path/ -r

# Copy OBS file to local
ma-cli obs-copy obs://bucket/path/ /local/path/ -r
```

Reference: [ModelArts CLI命令功能介绍](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/devtool-modelarts_0303.html)

---

## MoXing Data Operations

MoXing (mox) is a ModelArts utility library for transparent OBS file access, allowing code to treat OBS paths like local paths.

### Common MoXing Operations

```python
import moxing as mox

# Copy file from OBS to local
mox.file.copy("obs://bucket/data/train.csv", "/cache/train.csv")

# Copy directory from OBS to local
mox.file.copy_parallel("obs://bucket/dataset/", "/cache/dataset/")

# Copy local results back to OBS
mox.file.copy_parallel("/cache/output/", "obs://bucket/output/")

# List files in OBS directory
file_list = mox.file.list_directory("obs://bucket/data/")

# Check if file exists
exists = mox.file.exists("obs://bucket/model/checkpoint.pth")
```

Reference: [MoXing常用操作的样例代码](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_11_0005.html)

---

## Reference Documentation

- [Notebook使用场景](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/devtool-modelarts_0002.html)
- [管理Notebook实例](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/devtool-modelarts_0003.html)
- [JupyterLab常用功能介绍](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/devtool-modelarts_0013.html)
- [VS Code连接Notebook方式介绍](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/devtool-modelarts_0019.html)
- [MoXing Framework功能介绍](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_11_0001.html)
- [ModelArts CLI命令参考](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/devtool-modelarts_0301.html)
