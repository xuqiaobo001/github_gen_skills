---
name: ascend-training-reliability
description: Guide for configuring high-reliability training on Huawei Ascend NPU clusters. This skill should be used when users need to set up checkpoint-based resumable training, configure automatic fault recovery, detect and handle training hangs, or analyze training failure logs on Ascend NPU hardware.
license: Apache 2.0
---

# Ascend Training Reliability Guide

## Overview

This skill provides guidance for building highly reliable model training pipelines on Huawei Ascend NPU. It covers checkpoint-based resumable training, automatic hang detection and restart, fault tolerance configuration, and failure log analysis — all critical for long-running large model training jobs.

---

## When to Use

- Configuring checkpoint saving and resumable training (断点续训)
- Setting up automatic hang detection for NPU training jobs
- Configuring automatic restart on training failure
- Analyzing training failure logs on ModelArts Standard
- Designing fault-tolerant training strategies for large models

---

## Checkpoint-Based Resumable Training

### Basic Checkpoint Save/Load Pattern

```python
import os
import torch
import torch_npu

def save_checkpoint(model, optimizer, epoch, step, loss, path):
    """Save training checkpoint to OBS-accessible path."""
    checkpoint = {
        "model_state_dict": model.state_dict(),
        "optimizer_state_dict": optimizer.state_dict(),
        "epoch": epoch,
        "step": step,
        "loss": loss,
    }
    torch.save(checkpoint, path)
    print(f"Checkpoint saved: epoch={epoch}, step={step}")

def load_checkpoint(model, optimizer, path):
    """Load checkpoint and resume training state."""
    if not os.path.exists(path):
        print("No checkpoint found, starting from scratch.")
        return 0, 0
    checkpoint = torch.load(path, map_location="npu:0")
    model.load_state_dict(checkpoint["model_state_dict"])
    optimizer.load_state_dict(checkpoint["optimizer_state_dict"])
    epoch = checkpoint["epoch"]
    step = checkpoint["step"]
    print(f"Resumed from: epoch={epoch}, step={step}")
    return epoch, step
```

### Checkpoint Strategy for Large Models

For large models (tens of billions of parameters), use sharded checkpoints:

```python
from torch.distributed.checkpoint import save_state_dict, load_state_dict

# Save sharded checkpoint (each rank saves its own shard)
save_state_dict(
    state_dict={"model": model.state_dict()},
    storage_writer=FileSystemWriter(checkpoint_dir),
)

# Load sharded checkpoint
load_state_dict(
    state_dict={"model": model.state_dict()},
    storage_reader=FileSystemReader(checkpoint_dir),
)
```

### Checkpoint Save Frequency

Recommended strategy based on training scale:

| Training Scale | Save Interval | Storage Location |
|---|---|---|
| Small model (< 1B params) | Every epoch | Local + OBS |
| Medium model (1B-10B) | Every N steps (e.g., 500) | OBS parallel file system |
| Large model (> 10B) | Every N steps (e.g., 100-200) | OBS parallel file system |

Reference: [设置断点续训练](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/develop-modelarts-0023.html)

---

## Training Hang Detection

### What Causes Training Hangs on NPU

- HCCL collective communication deadlock (one rank fails, others wait)
- NPU hardware ECC error causing silent stall
- Data loader starvation (I/O bottleneck)
- Gradient synchronization timeout in DDP

### ModelArts Built-in Hang Detection

ModelArts Standard provides built-in hang detection. To enable:

1. In the training job configuration, enable **"训练作业卡死检测"**
2. Set the detection interval (e.g., 30 minutes of no log output)
3. Configure the action: alert only, or auto-restart

### Custom Hang Detection with Heartbeat

```python
import time
import threading

class HangDetector:
    def __init__(self, timeout_seconds=1800):
        self.timeout = timeout_seconds
        self.last_heartbeat = time.time()
        self._start_monitor()

    def heartbeat(self):
        """Call this in the training loop to signal progress."""
        self.last_heartbeat = time.time()

    def _start_monitor(self):
        def monitor():
            while True:
                elapsed = time.time() - self.last_heartbeat
                if elapsed > self.timeout:
                    print(f"HANG DETECTED: No heartbeat for {elapsed:.0f}s")
                    # Trigger recovery action
                    os._exit(1)  # Force exit to trigger auto-restart
                time.sleep(60)
        t = threading.Thread(target=monitor, daemon=True)
        t.start()

# Usage in training loop
detector = HangDetector(timeout_seconds=1800)
for step, batch in enumerate(train_loader):
    loss = train_step(model, batch)
    detector.heartbeat()
```

Reference: [训练作业卡死检测](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_trouble_0108.html)

---

## Automatic Restart Configuration

### Conditional Restart (on failure)

ModelArts supports automatic restart when training fails. Configure via:

- **Max restart count**: Number of retry attempts (e.g., 3)
- **Restart condition**: On non-zero exit code
- **Checkpoint path**: Must be on persistent storage (OBS) for restart to resume

### Unconditional Automatic Restart

For long-running pretraining jobs, enable unconditional restart:

- Training job restarts periodically regardless of failure
- Combined with checkpoint saving, this prevents resource waste from silent hangs

Reference: [设置无条件自动重启](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/develop-modelarts-0041.html)

---

## Fault Tolerance Checklist

### Pre-training Checklist

```python
# ModelArts training job configuration
fault_tolerance_config = {
    # Checkpoint
    "checkpoint_save_interval_steps": 200,
    "checkpoint_save_path": "/obs/bucket/checkpoints/",
    "checkpoint_max_keep": 5,

    # Hang detection
    "hang_detection_enabled": True,
    "hang_detection_timeout_minutes": 30,

    # Auto restart
    "auto_restart_enabled": True,
    "max_restart_count": 3,

    # Log collection
    "log_output_path": "/obs/bucket/logs/",
    "log_level": "INFO",
}
```

### Runtime Monitoring Points

| Metric | Normal Range | Alert Threshold |
|---|---|---|
| NPU utilization | 80-100% | < 50% for > 5 min |
| NPU memory usage | 60-95% | > 95% |
| Training loss | Decreasing trend | NaN or sudden spike |
| Step time | Stable | > 2x average |
| HCCL latency | < 100ms | > 500ms |

---

## Failure Log Analysis

### Common Error Patterns

| Log Pattern | Meaning | Action |
|---|---|---|
| `ECC error` | NPU hardware memory error | Auto-restart, report node for repair |
| `HCCL timeout` | Communication failure between nodes | Check network, increase timeout |
| `NPU out of memory` | Insufficient NPU HBM | Reduce batch size or model partition |
| `aicore kernel error` | Operator execution failure | Check operator compatibility |
| `Device disconnected` | NPU device lost | Node hardware issue, trigger failover |

### Log Collection Paths

```bash
# ModelArts training logs
/obs/{bucket}/logs/{job_id}/

# CANN runtime logs
/var/log/npu/slog/

# HCCL communication logs
/var/log/npu/hccl/
```

Reference: [训练日志失败分析](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/develop-modelarts-0098.html)

---

## Reference Documentation

- [模型训练高可靠性](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/develop-modelarts-0021.html)
- [训练作业容错检查](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_trouble_0003.html)
- [训练作业卡死重启](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/develop-modelarts-0043.html)
- [训练容器生命周期](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/develop-container-0001.html)
