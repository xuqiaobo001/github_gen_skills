---
name: lite-server-ad-inference-testing
description: Guide for automated autonomous driving model inference deployment and testing (performance, reliability, functionality, accuracy) on ModelArts Lite Server with Ascend NPU. Covers BEV perception, 3D detection, planning models. This skill should be used when users need to deploy and test autonomous driving inference on Lite Server.
license: Apache 2.0
---

# Lite Server Autonomous Driving Inference Deployment & Testing Guide

## Overview

End-to-end guidance for deploying and testing autonomous driving (AD) model inference on ModelArts Lite Server with Ascend NPU. Covers BEV perception, 3D object detection, point cloud processing, and end-to-end planning models, with systematic testing for performance, reliability, functionality, and accuracy.

---

## When to Use

- Deploying BEV perception models (BEVFormer, BEVDet) on Ascend NPU
- Deploying 3D object detection (PointPillars, CenterPoint, SECOND) on NPU
- Deploying end-to-end AD models (UniAD, VAD) on NPU
- Running AD inference performance benchmarks (FPS, latency per frame)
- Running AD reliability tests (continuous stream, sensor dropout handling)
- Running AD functionality tests (multi-sensor fusion, output format)
- Running AD accuracy tests (mAP, NDS comparison between GPU and NPU)

---

## Model & NPU Planning

| Model | Task | Input | Min NPU | HBM Estimate |
|---|---|---|---|---|
| BEVFormer | BEV Perception | Multi-cam images | 1× 910B | ~12 GB |
| BEVDet | BEV 3D Detection | Multi-cam images | 1× 910B | ~8 GB |
| PointPillars | 3D Detection | LiDAR point cloud | 1× 910B | ~4 GB |
| CenterPoint | 3D Detection | LiDAR point cloud | 1× 910B | ~6 GB |
| SECOND | 3D Detection | LiDAR point cloud | 1× 910B | ~4 GB |
| UniAD | End-to-End AD | Multi-cam + LiDAR | 2× 910B | ~24 GB |
| VAD | End-to-End Planning | Multi-cam images | 1× 910B | ~10 GB |

---

## Deployment

### ONNX Runtime on Ascend NPU

```bash
# Install ONNX Runtime with CANN backend
pip install onnxruntime-cann

# Export model to ONNX (example: BEVFormer)
python tools/export_onnx.py \
  --config configs/bevformer_base.py \
  --checkpoint /data/models/bevformer_base.pth \
  --output /data/models/bevformer_base.onnx
```

```python
#!/usr/bin/env python3
"""ad_serve.py — AD model inference server on NPU via ONNX Runtime"""
import numpy as np, onnxruntime as ort, json
from flask import Flask, request, jsonify

app = Flask(__name__)

# Load model with CANN Execution Provider
sess = ort.InferenceSession(
    "/data/models/bevformer_base.onnx",
    providers=["CANNExecutionProvider"]
)

@app.route("/predict", methods=["POST"])
def predict():
    data = request.json
    # Expect base64-encoded images or numpy arrays
    images = np.array(data["images"], dtype=np.float32)  # [N, C, H, W]
    outputs = sess.run(None, {"images": images})
    # outputs: bboxes, scores, labels
    results = {
        "bboxes_3d": outputs[0].tolist(),
        "scores": outputs[1].tolist(),
        "labels": outputs[2].tolist()
    }
    return jsonify(results)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

### PyTorch + torch_npu Direct Inference

```python
import torch, torch_npu
from mmdet3d.apis import init_model, inference_detector

config = "/data/configs/centerpoint_voxel.py"
checkpoint = "/data/models/centerpoint.pth"
model = init_model(config, checkpoint, device="npu:0")

# Inference on point cloud
result = inference_detector(model, "/data/test/lidar_sample.bin")
print(f"Detected {len(result.pred_instances_3d)} objects")
```

---

## Performance Testing

### Key Metrics

| Metric | Description | Target |
|---|---|---|
| FPS | Frames per second (single stream) | > 10 FPS (real-time threshold) |
| Latency | Per-frame inference time (ms) | < 100ms |
| HBM Peak | Peak NPU memory | < 80% HBM |
| Preprocessing | Data loading + transform time | < 20ms |
| Postprocessing | NMS + output formatting time | < 10ms |

### Benchmark Script

```python
#!/usr/bin/env python3
"""ad_perf_bench.py — AD model inference performance benchmark"""
import time, numpy as np, statistics

def benchmark_onnx(model_path, num_frames=200):
    import onnxruntime as ort
    sess = ort.InferenceSession(model_path, providers=["CANNExecutionProvider"])
    input_name = sess.get_inputs()[0].name
    input_shape = sess.get_inputs()[0].shape
    dummy = np.random.randn(*[s if isinstance(s,int) else 1 for s in input_shape]).astype(np.float32)

    # Warmup
    for _ in range(10):
        sess.run(None, {input_name: dummy})

    latencies = []
    for _ in range(num_frames):
        t0 = time.perf_counter()
        sess.run(None, {input_name: dummy})
        latencies.append((time.perf_counter() - t0) * 1000)

    print(f"Frames: {num_frames}")
    print(f"Latency — P50: {statistics.median(latencies):.1f}ms  P99: {sorted(latencies)[int(len(latencies)*0.99)]:.1f}ms")
    print(f"FPS: {1000/statistics.mean(latencies):.1f}")

if __name__ == "__main__":
    benchmark_onnx("/data/models/bevformer_base.onnx")
```

---

## Reliability Testing

### Continuous Stream Stability

```bash
#!/bin/bash
# ad_stability_test.sh — Simulate continuous sensor stream for N hours
DURATION_HOURS=${1:-8}
API="http://localhost:8000/predict"
LOG="/var/log/ad_stability_$(date +%Y%m%d_%H%M%S).log"
END=$(($(date +%s) + DURATION_HOURS * 3600))
OK=0; NG=0; FRAME=0

# Generate dummy input payload (6 camera images, 3×900×1600)
PAYLOAD=$(python3 -c "
import json, numpy as np
imgs = np.random.randn(6,3,900,1600).astype('float32')
print(json.dumps({'images': imgs[:,:,:100,:100].tolist()}))
")

echo "AD stability test: ${DURATION_HOURS}h" | tee "$LOG"
while [ $(date +%s) -lt $END ]; do
  FRAME=$((FRAME+1))
  CODE=$(curl -s -o /dev/null -w "%{http_code}" "$API" \
    -H "Content-Type: application/json" -d "$PAYLOAD" --max-time 30)
  if [ "$CODE" = "200" ]; then OK=$((OK+1))
  else NG=$((NG+1)); echo "[$(date)] FAIL frame=$FRAME http=$CODE" >> "$LOG"; fi
  sleep 0.1  # ~10 FPS simulation
done
echo "Done: ok=$OK fail=$NG total=$FRAME" | tee -a "$LOG"
```

### Fault Scenarios

| Test Case | Method | Expected |
|---|---|---|
| Corrupted input | Send NaN/Inf in image array | Returns error, service stays alive |
| Missing sensor | Send 5 of 6 cameras | Graceful degradation or clear error |
| Oversized point cloud | 10× normal point count | OOM handled, returns error |
| Process crash recovery | Kill and restart service | Recovers within 30s |

---

## Functionality Testing

| Test Case | Input | Validation |
|---|---|---|
| Single-frame detection | 1 LiDAR frame | Returns bboxes_3d, scores, labels |
| Multi-camera BEV | 6 camera images | Returns BEV feature map + 3D boxes |
| Batch inference | 4 frames batch | Returns results for each frame |
| Score threshold | confidence_threshold=0.5 | Only high-confidence detections |
| Class filtering | filter_classes=["car","pedestrian"] | Only specified classes returned |
| Output coordinate system | — | Boxes in ego-vehicle coordinate |
| Empty scene | Scene with no objects | Returns empty detection list |
| Dense scene | 200+ objects | All objects detected, no crash |

---

## Accuracy Testing

### GPU vs NPU mAP/NDS Comparison

```python
#!/usr/bin/env python3
"""ad_accuracy_test.py — Compare NPU vs GPU detection accuracy on nuScenes val"""
import json, numpy as np

def load_results(path):
    with open(path) as f:
        return json.load(f)

def compute_ap(gt_boxes, pred_boxes, iou_threshold=0.5):
    """Simplified 3D IoU AP computation"""
    # In practice, use nuscenes-devkit or mmdet3d evaluation
    # This is a placeholder for the evaluation pipeline
    pass

# Step 1: Run inference on validation set (both GPU and NPU)
# python eval.py --device gpu --output /data/results/gpu_results.json
# python eval.py --device npu --output /data/results/npu_results.json

# Step 2: Compare metrics
gpu_results = load_results("/data/results/gpu_results.json")
npu_results = load_results("/data/results/npu_results.json")

print("=== AD Accuracy: GPU vs NPU ===")
print(f"GPU — mAP: {gpu_results['mAP']:.4f}  NDS: {gpu_results['NDS']:.4f}")
print(f"NPU — mAP: {npu_results['mAP']:.4f}  NDS: {npu_results['NDS']:.4f}")
print(f"mAP diff: {abs(gpu_results['mAP']-npu_results['mAP']):.4f}")
print(f"NDS diff: {abs(gpu_results['NDS']-npu_results['NDS']):.4f}")
```

### Evaluation with nuScenes DevKit

```bash
# Run official nuScenes evaluation on NPU inference results
python -m nuscenes.eval.detection.evaluate \
  --result_path /data/results/npu_results.json \
  --eval_set val \
  --dataroot /data/nuscenes \
  --output_dir /data/eval_output
```

> mAP 差异 < 0.5% 且 NDS 差异 < 0.5% 视为精度对齐通过。

---

## Reference Documentation

- [LLM/AIGC等模型基于Lite Server适配NPU的训练推理指导](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0014.html)
- [NPU服务器上配置Lite Server资源软件环境](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0011.html)
- [Lite Server算力资源和镜像版本配套关系](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0004.html)
- [Lite Server节点一键式压测](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0035.html)
- [Lite Server节点故障诊断](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0031.html)
