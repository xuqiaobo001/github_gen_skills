---
name: lite-server-aigc-inference-testing
description: Guide for automated AIGC model inference deployment and testing (performance, reliability, functionality, accuracy) on ModelArts Lite Server with Ascend NPU. Covers Stable Diffusion, SDXL, DiT and similar image/video generation models. This skill should be used when users need to deploy and test AIGC inference services on Lite Server.
license: Apache 2.0
---

# Lite Server AIGC Inference Deployment & Testing Guide

## Overview

End-to-end guidance for deploying and testing AIGC (AI-Generated Content) inference services on ModelArts Lite Server with Ascend NPU. Covers image generation (Stable Diffusion, SDXL, DiT), video generation, and systematic testing for performance, reliability, functionality, and accuracy.

---

## When to Use

- Deploying Stable Diffusion / SDXL / DiT inference on Ascend NPU
- Deploying image/video generation API services on Lite Server
- Running AIGC performance benchmarks (latency per image, throughput)
- Running AIGC reliability tests (long-duration, memory leak, batch stability)
- Running AIGC functionality tests (resolution, LoRA, ControlNet, img2img)
- Running AIGC accuracy tests (GPU vs NPU image quality comparison)

---

## Model & NPU Planning

| Model | Type | Min NPU (FP16) | HBM Estimate |
|---|---|---|---|
| Stable Diffusion 1.5 | Text-to-Image | 1× 910B | ~8 GB |
| SDXL 1.0 | Text-to-Image | 1× 910B | ~16 GB |
| DiT-XL/2 | Text-to-Image (Transformer) | 1× 910B | ~12 GB |
| Stable Video Diffusion | Image-to-Video | 2× 910B | ~32 GB |
| FLUX.1 | Text-to-Image | 2× 910B | ~24 GB |

---

## Deployment

### Stable Diffusion via Diffusers + torch_npu

```python
#!/usr/bin/env python3
"""sd_serve.py — Stable Diffusion inference server on NPU"""
import torch, torch_npu
from diffusers import StableDiffusionPipeline
from flask import Flask, request, send_file
import io

app = Flask(__name__)
pipe = StableDiffusionPipeline.from_pretrained(
    "/data/models/stable-diffusion-v1-5", torch_dtype=torch.float16
).to("npu:0")

@app.route("/generate", methods=["POST"])
def generate():
    data = request.json
    prompt = data.get("prompt", "a cat")
    steps = data.get("steps", 30)
    width = data.get("width", 512)
    height = data.get("height", 512)
    image = pipe(prompt, num_inference_steps=steps,
                 width=width, height=height).images[0]
    buf = io.BytesIO()
    image.save(buf, format="PNG")
    buf.seek(0)
    return send_file(buf, mimetype="image/png")

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

### Launch Script

```bash
#!/bin/bash
# deploy_aigc.sh
MODEL_PATH=${1:?Usage: deploy_aigc.sh <model_path> [port]}
PORT=${2:-8000}

npu-smi info > /dev/null 2>&1 || { echo "NPU not available"; exit 1; }

echo "Deploying AIGC model: $MODEL_PATH on port $PORT"
nohup python3 sd_serve.py > /var/log/aigc_serve.log 2>&1 &
echo "PID: $!  Log: /var/log/aigc_serve.log"
```

---

## Performance Testing

### Key Metrics

| Metric | Description | Target (SD 1.5, 512×512) |
|---|---|---|
| Latency | Time per image generation | < 5s (30 steps) |
| Throughput | Images per minute | > 12 img/min |
| HBM Peak | Peak NPU memory during generation | < 80% HBM |
| Batch Latency | Time for N images in one call | Near-linear scaling |

### Benchmark Script

```python
#!/usr/bin/env python3
"""aigc_perf_bench.py — AIGC inference performance benchmark"""
import time, requests, statistics, concurrent.futures

API = "http://localhost:8000/generate"
PROMPTS = [
    "a beautiful sunset over mountains, oil painting",
    "a futuristic city skyline at night, cyberpunk",
    "a white cat sitting on a windowsill, watercolor",
    "an astronaut riding a horse on mars, photorealistic",
]

def single_request(prompt, steps=30, width=512, height=512):
    t0 = time.perf_counter()
    r = requests.post(API, json={
        "prompt": prompt, "steps": steps,
        "width": width, "height": height
    }, timeout=120)
    elapsed = time.perf_counter() - t0
    return {"latency": elapsed, "status": r.status_code, "size": len(r.content)}

def run_benchmark(num_requests=20, concurrency=1):
    results = []
    with concurrent.futures.ThreadPoolExecutor(max_workers=concurrency) as pool:
        futures = [pool.submit(single_request, PROMPTS[i % len(PROMPTS)])
                   for i in range(num_requests)]
        for f in concurrent.futures.as_completed(futures):
            results.append(f.result())
    lats = [r["latency"] for r in results if r["status"] == 200]
    print(f"Concurrency: {concurrency}, Requests: {num_requests}")
    print(f"Latency — P50: {statistics.median(lats):.2f}s  P99: {sorted(lats)[int(len(lats)*0.99)]:.2f}s")
    print(f"Throughput: {len(lats)/sum(lats)*60:.1f} img/min")
    print(f"Success rate: {len(lats)}/{num_requests}")

if __name__ == "__main__":
    run_benchmark()
```

---

## Reliability Testing

### Long-Duration Stability Test

```bash
#!/bin/bash
# aigc_stability_test.sh — Continuous image generation for N hours
DURATION_HOURS=${1:-12}
API="http://localhost:8000/generate"
LOG="/var/log/aigc_stability_$(date +%Y%m%d_%H%M%S).log"
END=$(($(date +%s) + DURATION_HOURS * 3600))
OK=0; NG=0; ROUND=0

echo "AIGC stability test: ${DURATION_HOURS}h" | tee "$LOG"
while [ $(date +%s) -lt $END ]; do
  ROUND=$((ROUND+1))
  CODE=$(curl -s -o /tmp/aigc_out.png -w "%{http_code}" "$API" \
    -H "Content-Type: application/json" \
    -d "{\"prompt\":\"test image round $ROUND\",\"steps\":20}")
  SIZE=$(stat -c%s /tmp/aigc_out.png 2>/dev/null || echo 0)
  if [ "$CODE" = "200" ] && [ "$SIZE" -gt 1000 ]; then
    OK=$((OK+1))
  else
    NG=$((NG+1))
    echo "[$(date)] FAIL round=$ROUND http=$CODE size=$SIZE" >> "$LOG"
  fi
  sleep 5
done
echo "Done: ok=$OK fail=$NG total=$ROUND" | tee -a "$LOG"
```

### Memory Leak Detection

```bash
# Monitor HBM usage over time — should remain stable
for i in $(seq 1 100); do
  curl -s -o /dev/null http://localhost:8000/generate \
    -d '{"prompt":"memory test","steps":20}'
  echo "$(date +%s) $(npu-smi info -t memory -i 0 | grep Used)" >> /var/log/aigc_hbm.log
done
# Check: HBM Used should not continuously increase
```

---

## Functionality Testing

| Test Case | Input | Validation |
|---|---|---|
| Text-to-Image basic | prompt + default params | Returns valid PNG, correct resolution |
| Custom resolution | width=768, height=1024 | Output matches requested size |
| Negative prompt | prompt + negative_prompt | Output avoids negative content |
| Batch generation | num_images=4 | Returns 4 distinct images |
| Seed reproducibility | Same prompt + seed twice | Two outputs are pixel-identical |
| Step variation | steps=10 / 30 / 50 | All return valid images, quality improves |
| LoRA loading | prompt + lora_path | Style matches LoRA training |
| ControlNet | prompt + control_image | Output follows control structure |

---

## Accuracy Testing

### GPU vs NPU Image Quality Comparison

```python
#!/usr/bin/env python3
"""aigc_accuracy_test.py — Compare NPU vs GPU generated images"""
import requests, numpy as np
from PIL import Image
from skimage.metrics import structural_similarity as ssim
import io

GPU_API = "http://<gpu-server>:8000/generate"
NPU_API = "http://localhost:8000/generate"

PROMPTS = [
    "a red rose on a white background",
    "a mountain landscape with snow",
    "a portrait of a robot, digital art",
]

def fetch_image(api, prompt, seed=42):
    r = requests.post(api, json={"prompt": prompt, "steps": 30, "seed": seed}, timeout=120)
    return Image.open(io.BytesIO(r.content)).convert("RGB")

print("=== AIGC Accuracy: GPU vs NPU ===")
for p in PROMPTS:
    gpu_img = np.array(fetch_image(GPU_API, p))
    npu_img = np.array(fetch_image(NPU_API, p))
    score = ssim(gpu_img, npu_img, channel_axis=2)
    print(f"Prompt: {p[:40]}...  SSIM: {score:.4f}  {'PASS' if score > 0.85 else 'WARN'}")
```

> SSIM > 0.90 表示高度一致，0.85–0.90 可接受，< 0.85 需排查精度问题。

---

## Reference Documentation

- [LLM/AIGC等模型基于Lite Server适配NPU的训练推理指导](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0014.html)
- [NPU服务器上配置Lite Server资源软件环境](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0011.html)
- [Lite Server节点一键式压测](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0035.html)
- [Lite Server节点故障诊断](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0031.html)
