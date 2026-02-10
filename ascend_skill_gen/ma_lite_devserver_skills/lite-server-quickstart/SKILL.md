---
name: lite-server-quickstart
description: Guide for getting started with ModelArts Lite Server, including usage prerequisites, workflow overview, high-risk operations, compute-image compatibility, and resource provisioning. This skill should be used when users need to understand Lite Server basics, check hardware-software compatibility, or provision Lite Server resources.
license: Apache 2.0
---

# Lite Server Quick Start Guide

## Overview

This skill provides guidance for getting started with Huawei ModelArts Lite Server. It covers the overall usage workflow, high-risk operation warnings, compute resource and image version compatibility, and step-by-step resource provisioning procedures.

---

## When to Use

- First time using ModelArts Lite Server
- Understanding the end-to-end Lite Server usage workflow
- Checking which image versions are compatible with specific compute resources
- Reviewing high-risk operations before performing administrative tasks
- Provisioning new Lite Server resources (new or legacy console)

---

## Lite Server Usage Workflow

### End-to-End Flow

```
1. Resource Provisioning (开通资源)
   └── Select compute specification (NPU/GPU)
       └── Choose billing mode (yearly/monthly)

2. Resource Configuration (配置资源)
   ├── Configure network (VPC, subnet, security group)
   ├── Configure storage (EVS, SFS)
   └── Configure software environment (optional)

3. Resource Usage (使用资源)
   ├── SSH login to server
   ├── Upload data and code
   └── Run training / inference tasks

4. Resource Management (管理资源)
   ├── Start / Stop / Restart server
   ├── Monitor resource utilization
   └── Manage OS images
```

Reference: [Lite Server使用流程](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0002.html)

---

## High-Risk Operations

The following operations may cause service interruption or data loss. Exercise caution:

| Operation | Risk Level | Impact |
|---|---|---|
| 切换/重置操作系统 | High | Data on system disk will be erased |
| 关机 | High | Running tasks will be terminated |
| 重启 | Medium | Running tasks will be interrupted |
| 升级驱动固件 | Medium | Requires server restart, tasks interrupted |
| 退订资源 | Critical | All data and configurations permanently deleted |

### Precautions

- Always back up important data before performing high-risk operations
- Ensure no critical training jobs are running before server restart or shutdown
- Verify driver/firmware compatibility before upgrading

Reference: [Lite Server高危操作一览表](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0003.html)

---

## Compute Resource and Image Compatibility

### Hardware-Software Matching Rule

```
Compute Specification (NPU/GPU model)
  └── Supported OS images
       └── Pre-installed driver/firmware versions
            └── Compatible CANN / CUDA versions
```

### Common Compute Specifications

| Specification | Accelerator | Typical Use Case |
|---|---|---|
| Ascend 910A | NPU | Medium-scale training |
| Ascend 910B | NPU | Large-scale LLM training |
| Ascend 910C | NPU | Next-gen large model training |
| NVIDIA V100 | GPU | General training/inference |
| NVIDIA A100 | GPU | Large model training |

### How to Check Compatibility

1. Navigate to ModelArts Console → Lite Server
2. Check the resource specification details page
3. Verify the supported image list for your compute specification
4. Ensure driver/firmware versions match the image requirements

Reference: [Lite Server算力资源和镜像版本配套关系](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0004.html)

---

## Resource Provisioning

### New Console Provisioning

Steps to provision Lite Server resources via the new console:

1. Log in to Huawei Cloud Console → ModelArts → Lite Server
2. Click "Purchase Lite Server"
3. Configure the following parameters:

| Parameter | Description |
|---|---|
| Region | Select the deployment region |
| Compute Specification | Choose NPU or GPU flavor |
| Billing Mode | Yearly/Monthly subscription |
| Quantity | Number of servers |
| OS Image | Select compatible image |
| Network | Configure VPC and subnet |
| Storage | Configure system and data disks |

4. Confirm configuration and submit order
5. Wait for resource provisioning to complete

Reference: [Lite Server资源开通（新版页面）](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0005.html)

### Legacy Console Provisioning

For users still using the legacy console interface, the provisioning flow is similar but with a different UI layout.

Reference: [Lite Server资源开通（旧版页面）](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0052.html)

---

## Reference Documentation

- [ModelArts Lite Server用户指南](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0001.html)
- [Lite Server使用前必读](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0001.html)
- [Lite Server使用流程](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0002.html)
- [Lite Server高危操作一览表](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0003.html)
- [Lite Server算力资源和镜像版本配套关系](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0004.html)
- [Lite Server资源开通（新版页面）](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0005.html)
- [Lite Server资源开通（旧版页面）](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0052.html)
