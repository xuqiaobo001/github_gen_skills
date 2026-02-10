---
name: lite-server-lifecycle
description: Guide for managing the full lifecycle of ModelArts Lite Server instances, including viewing details, power on/off, status sync, OS switch/reset/creation, hot standby, renaming, authorized repair, and restart. This skill should be used when users need to perform server administration tasks on Lite Server.
license: Apache 2.0
---

# Lite Server Lifecycle Management Guide

## Overview

This skill provides guidance for managing the full lifecycle of ModelArts Lite Server instances, covering server status viewing, power management, OS operations, hot standby configuration, and node repair authorization.

---

## When to Use

- Viewing Lite Server details and status
- Starting, stopping, or restarting Lite Server instances
- Synchronizing server status after maintenance
- Switching, resetting, or creating custom OS images
- Configuring hot standby for high availability
- Renaming servers or authorizing node repair

---

## Server Status Overview

### Server States

| State | Description | Allowed Operations |
|---|---|---|
| Running | Server is active and accessible | Stop, Restart, Sync |
| Stopped | Server is powered off | Start, Switch OS, Reset OS |
| Starting | Server is booting up | Wait |
| Stopping | Server is shutting down | Wait |
| Error | Server encountered a fault | Repair, Restart |
| Upgrading | Driver/firmware upgrade in progress | Wait |

### Viewing Server Details

Navigate to ModelArts Console → Lite Server to view:

- Server name and ID
- Compute specification (NPU/GPU model)
- Network configuration (VPC, subnet, IP)
- Storage configuration (disk type and size)
- OS image version
- Running status and uptime

Reference: [查看Lite Server服务器详情](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0017.html)

---

## Power Management

### Start / Stop Server

| Operation | Precondition | Impact |
|---|---|---|
| Start (开机) | Server is in Stopped state | Server boots with current OS image |
| Stop (关机) | Server is in Running state | All running processes terminated, data on system disk preserved |

**Warning**: Stopping a server will immediately terminate all running training jobs. Save checkpoints before stopping.

Reference: [开机或关机Lite Server服务器](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0018.html)

### Restart Server

Restart performs a graceful reboot:

1. Running processes receive shutdown signal
2. OS shuts down cleanly
3. Server boots back up automatically

Use restart when:
- Server becomes unresponsive
- After software environment changes that require reboot
- After kernel or driver updates

Reference: [重启Lite Server服务器](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0053.html)

### Sync Server Status

When the console status does not reflect the actual server state (e.g., after backend maintenance), use the sync operation to refresh:

1. Navigate to server details page
2. Click "Sync Status"
3. System queries actual server state and updates console display

Reference: [同步Lite Server服务器状态](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0019.html)

---

## OS Management

### Switch or Reset OS

| Operation | Description | Data Impact |
|---|---|---|
| Switch OS | Change to a different OS image | System disk data erased |
| Reset OS | Reinstall current OS image | System disk data erased |

**Warning**: Both operations erase all data on the system disk. Back up important data to data disks or OBS before proceeding.

Steps:
1. Stop the server
2. Navigate to server details → "Switch/Reset OS"
3. Select target OS image
4. Confirm and wait for operation to complete

Reference: [切换或重置Lite Server服务器操作系统](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0020.html)

### Create Custom OS Image

Create a reusable OS image from a configured server:

1. Install and configure all required software on the server
2. Stop the server
3. Navigate to server details → "Create Image"
4. Provide image name and description
5. Wait for image creation to complete

Use cases:
- Standardize team development environments
- Pre-install ML frameworks and dependencies
- Create recovery snapshots before risky operations

Reference: [制作Lite Server服务器操作系统](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0030.html)

---

## Hot Standby Management

Hot standby provides high availability by maintaining backup resources that can quickly replace faulty nodes:

- Automatically switches to standby node when primary node fails
- Minimizes training interruption during hardware failures
- Requires pre-configured standby resources in the same resource pool

Reference: [Lite Server资源热备管理](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0046.html)

---

## Other Management Operations

### Rename Server

Modify the display name of a Lite Server instance for easier identification:

1. Navigate to server details page
2. Click the edit icon next to the server name
3. Enter the new name and confirm

Reference: [修改Lite Server名称](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0051.html)

### Authorize Node Repair

When a node encounters hardware faults that require physical intervention:

1. System detects the fault and notifies the administrator
2. Administrator reviews the fault details
3. Authorize the repair operation via the console
4. Huawei Cloud operations team performs the repair
5. Node is restored and rejoins the server

Reference: [授权修复Lite Server节点](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0039.html)

---

## Reference Documentation

- [Lite Server资源管理](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0016.html)
- [查看Lite Server服务器详情](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0017.html)
- [开机或关机Lite Server服务器](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0018.html)
- [同步Lite Server服务器状态](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0019.html)
- [切换或重置Lite Server服务器操作系统](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0020.html)
- [制作Lite Server服务器操作系统](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0030.html)
- [Lite Server资源热备管理](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0046.html)
- [修改Lite Server名称](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0051.html)
- [授权修复Lite Server节点](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0039.html)
- [重启Lite Server服务器](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0053.html)
