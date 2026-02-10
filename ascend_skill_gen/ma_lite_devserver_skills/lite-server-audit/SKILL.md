---
name: lite-server-audit
description: Guide for Lite Server audit, CloudPond NPU resource management, and resource unsubscription. This skill should be used when users need to audit Lite Server operations via CTS, manage CloudPond NPU resources, or unsubscribe Lite Server resources.
license: Apache 2.0
---

# Lite Server Audit & Other Management Guide

## Overview

This skill provides guidance for auditing Lite Server operations using CTS (Cloud Trace Service), managing CloudPond NPU resources via Lite Server, and unsubscribing Lite Server resources.

---

## When to Use

- Auditing Lite Server service operations via CTS
- Managing CloudPond NPU resources through Lite Server
- Unsubscribing and releasing Lite Server resources

---

## CloudPond NPU Resource Management

### What is CloudPond

CloudPond is Huawei Cloud's edge computing service that extends cloud capabilities to customer premises. Lite Server can manage NPU resources deployed on CloudPond infrastructure.

### Key Operations

| Operation | Description |
|---|---|
| View NPU resources | List all CloudPond NPU nodes managed by Lite Server |
| Monitor utilization | View NPU usage metrics on CloudPond nodes |
| Manage lifecycle | Start, stop, restart CloudPond NPU nodes |
| Configure network | Set up connectivity between CloudPond and cloud VPC |

### Use Cases

- Running AI inference at the edge with low latency
- Training models on-premises with CloudPond NPU resources
- Hybrid cloud AI workloads spanning cloud and edge

Reference: [Lite Server管理CloudPond的NPU资源](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0048.html)

---

## CTS Audit

### What is CTS

CTS (Cloud Trace Service) records all API operations performed on Lite Server resources for compliance auditing and security analysis.

### Audited Operations

| Operation Type | Examples |
|---|---|
| Resource provisioning | Create, delete Lite Server |
| Power management | Start, stop, restart server |
| OS management | Switch OS, reset OS, create image |
| Plugin management | Install plugin, upgrade driver |
| Configuration changes | Modify name, configure network |

### Viewing Audit Logs

1. Navigate to CTS Console → Trace List
2. Filter by service: "ModelArts"
3. Filter by resource type: "LiteServer"
4. Select time range
5. View trace details:
   - Operator (IAM user or agency)
   - Operation name and result
   - Timestamp
   - Source IP address
   - Request and response details

Reference: [使用CTS审计Lite Server服务操作](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0050.html)

---

## Resource Unsubscription

### Overview

When Lite Server resources are no longer needed, unsubscribe to stop billing and release resources.

### Pre-Unsubscription Checklist

1. **Back up all data** — system disk, data disk, and SFS data
2. **Export model checkpoints** to OBS or external storage
3. **Stop all running jobs** — training, inference, notebooks
4. **Record configuration** — save network, storage, and software configs for future reference
5. **Remove dependencies** — detach shared file systems, release EIPs

### Unsubscription Steps

1. Navigate to ModelArts Console → Lite Server
2. Select the target server
3. Click "Unsubscribe"
4. Confirm the unsubscription
5. System releases all associated resources

**Warning**: Unsubscription is irreversible. All data on the server will be permanently deleted.

Reference: [退订Lite Server资源](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0025.html)

---

## Reference Documentation

- [Lite Server管理CloudPond的NPU资源](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0048.html)
- [使用CTS审计Lite Server服务操作](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0050.html)
- [退订Lite Server资源](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0025.html)
