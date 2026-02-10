---
name: lite-server-network-storage
description: Guide for configuring Lite Server network, storage, and software environments on ModelArts. This skill should be used when users need to set up VPC networking, configure EVS/SFS storage, or install software environments including NPU server-specific configurations.
license: Apache 2.0
---

# Lite Server Network & Storage Configuration Guide

## Overview

This skill provides guidance for configuring Lite Server resources after provisioning, including network setup (VPC, subnet, security groups), storage configuration (EVS, SFS), and optional software environment installation for both general and NPU-specific scenarios.

---

## When to Use

- Configuring Lite Server network (VPC, subnet, security group, EIP)
- Setting up storage volumes (system disk, data disk, shared file system)
- Installing or configuring software environments on Lite Server
- Configuring NPU server-specific software (CANN, driver, firmware)
- Following the end-to-end resource configuration workflow

---

## Resource Configuration Workflow

```
Resource Configuration
├── 1. Configure Network
│   ├── VPC and Subnet
│   ├── Security Group rules
│   └── EIP (optional, for public access)
├── 2. Configure Storage
│   ├── System disk (EVS)
│   ├── Data disk (EVS)
│   └── Shared file system (SFS Turbo)
└── 3. Configure Software Environment (optional)
    ├── General software packages
    └── NPU-specific: CANN toolkit, driver, firmware
```

Reference: [Lite Server资源配置流程](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0007.html)

---

## Network Configuration

### VPC and Subnet

Lite Server must be associated with a VPC (Virtual Private Cloud) for network isolation and communication:

| Component | Description | Recommendation |
|---|---|---|
| VPC | Isolated virtual network | One VPC per project or team |
| Subnet | IP address segment within VPC | Ensure sufficient IP addresses |
| Security Group | Firewall rules for inbound/outbound | Open SSH (22), training ports as needed |
| EIP | Elastic IP for public access | Optional, for external data download |

### Key Considerations

- Multi-node training requires all servers in the same VPC and subnet
- Security group must allow inter-node communication for distributed training (HCCL ports)
- EIP is needed if training scripts download data from the internet

Reference: [配置Lite Server网络](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0008.html)

---

## Storage Configuration

### Storage Types

| Storage Type | Use Case | Characteristics |
|---|---|---|
| System Disk (EVS) | OS and system software | Fixed at provisioning, lost on OS reset |
| Data Disk (EVS) | Training data, model checkpoints | Persistent, survives server restart |
| SFS Turbo | Shared data across multiple servers | High throughput, multi-node access |

### Best Practices

- Store training data on data disks or SFS Turbo, never on system disk
- Use SFS Turbo for multi-node distributed training to share datasets
- Regularly back up important checkpoints to OBS (Object Storage Service)
- Monitor disk usage to avoid training failures due to full disk

### Mount Commands

```bash
# Check mounted disks
lsblk

# Mount a data disk (example)
mkdir -p /data
mount /dev/vdb1 /data

# Mount SFS Turbo (example)
mount -t nfs -o vers=3,timeo=600,noresvport,nolock {sfs_ip}:/ /mnt/sfs
```

Reference: [配置Lite Server存储](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0009.html)

---

## Software Environment Configuration (Optional)

### General Software Environment

After provisioning, you may need to install additional software packages:

```bash
# Update package manager
apt-get update

# Install common ML dependencies
pip install torch torchvision transformers accelerate

# Install system tools
apt-get install -y htop tmux git wget
```

Reference: [配置Lite Server软件环境（可选）](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0010.html)

### NPU Server Software Environment

For NPU (Ascend) servers, additional software configuration is required:

| Component | Description | Check Command |
|---|---|---|
| Ascend Driver | NPU hardware driver | `npu-smi info` |
| Ascend Firmware | NPU firmware | `npu-smi info -t board` |
| CANN Toolkit | Ascend compute library | `cat /usr/local/Ascend/ascend-toolkit/latest/version.cfg` |
| torch_npu | PyTorch NPU adapter | `python -c "import torch_npu; print(torch_npu.__version__)"` |

### NPU Environment Verification

```bash
# Verify NPU device availability
npu-smi info

# Verify PyTorch NPU integration
python -c "
import torch
import torch_npu
print(f'NPU available: {torch.npu.is_available()}')
print(f'NPU count: {torch.npu.device_count()}')
"
```

Reference: [NPU服务器上配置Lite Server资源软件环境](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0011.html)

---

## Reference Documentation

- [Lite Server资源配置](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0006.html)
- [Lite Server资源配置流程](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0007.html)
- [配置Lite Server网络](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0008.html)
- [配置Lite Server存储](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0009.html)
- [配置Lite Server软件环境（可选）](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0010.html)
- [NPU服务器上配置Lite Server资源软件环境](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0011.html)
