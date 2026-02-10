---
name: lite-server-plugin
description: Guide for managing plugins on ModelArts Lite Server, including AI plugin installation, Ascend driver/firmware upgrades, and CES Agent installation/upgrade. This skill should be used when users need to install, upgrade, or troubleshoot plugins on Lite Server instances.
license: Apache 2.0
---

# Lite Server Plugin Management Guide

## Overview

This skill provides guidance for managing plugins on ModelArts Lite Server, covering AI plugin installation, Ascend NPU driver and firmware upgrades, and CES (Cloud Eye Service) Agent plugin management.

---

## When to Use

- Installing AI plugins on Lite Server
- Upgrading Ascend NPU driver or firmware versions
- Installing or upgrading CES Agent for monitoring
- Troubleshooting plugin installation failures
- Checking plugin version compatibility

---

## Plugin Overview

| Plugin | Purpose | Applicable Hardware |
|---|---|---|
| AI Plugin | NPU compute libraries and tools | Ascend NPU |
| Ascend Driver/Firmware | Hardware driver and firmware | Ascend NPU |
| CES Agent | Cloud Eye monitoring agent | NPU and GPU |

---

## AI Plugin Installation

### What is the AI Plugin

The AI plugin provides essential NPU compute capabilities including:

- CANN (Compute Architecture for Neural Networks) toolkit
- NPU operator libraries
- npu-smi monitoring tool
- HCCL communication library

### Installation Steps

1. Navigate to ModelArts Console → Lite Server → Plugin Management
2. Select the target server
3. Choose "Install AI Plugin"
4. Select the plugin version compatible with your hardware
5. Confirm and wait for installation to complete

### Verify Installation

```bash
# Check NPU device status
npu-smi info

# Check CANN version
cat /usr/local/Ascend/ascend-toolkit/latest/version.cfg

# Verify HCCL availability
ls /usr/local/Ascend/ascend-toolkit/latest/lib64/libhccl.so
```

Reference: [安装Lite Server AI插件](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0032.html)

---

## Ascend Driver/Firmware Upgrade

### Version Compatibility Chain

```
NPU Hardware Model (910A/910B/910C)
  └── Driver version
       └── Firmware version
            └── CANN version
                 └── torch_npu version
```

All components must be version-compatible. Mismatched versions cause runtime errors.

### Pre-Upgrade Checklist

1. Back up important data and checkpoints
2. Stop all running training/inference jobs
3. Verify target version compatibility with current CANN and framework
4. Ensure server is in Running state

### Upgrade Steps

1. Navigate to Lite Server → Plugin Management
2. Select "Upgrade Driver/Firmware"
3. Choose target driver and firmware version
4. Confirm the upgrade
5. Server will restart automatically during upgrade
6. Verify after restart:

```bash
# Check new driver version
npu-smi info
cat /usr/local/Ascend/driver/version.info
```

**Warning**: Driver upgrade requires server restart. All running processes will be terminated.

Reference: [升级Lite Server中的Ascend驱动固件版本](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0033.html)

---

## CES Agent Plugin

### What is CES Agent

CES (Cloud Eye Service) Agent collects system-level and NPU-level metrics from Lite Server and reports them to the CES platform for monitoring and alerting.

Collected metrics include:
- CPU utilization, memory usage, disk I/O
- NPU utilization, HBM usage, NPU temperature
- Network throughput

### Install / Upgrade CES Agent

1. Navigate to Lite Server → Plugin Management
2. Select "Install/Upgrade CES Agent"
3. Choose the target version
4. Confirm installation
5. Verify agent status:

```bash
# Check CES Agent process
systemctl status cesagent

# View CES Agent logs
journalctl -u cesagent --tail 50
```

Reference: [安装/升级Lite Server中的CES Agent插件](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0054.html)

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| Plugin install fails | Network connectivity issue | Check EIP and security group rules |
| Driver version mismatch | Incompatible driver/CANN combo | Check version compatibility matrix |
| CES Agent not reporting | Agent process crashed | Restart cesagent service |
| npu-smi not found | AI plugin not installed | Install AI plugin first |

---

## Reference Documentation

- [Lite Server插件管理](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0045.html)
- [安装Lite Server AI插件](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0032.html)
- [升级Lite Server中的Ascend驱动固件版本](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0033.html)
- [安装/升级Lite Server中的CES Agent插件](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0054.html)
