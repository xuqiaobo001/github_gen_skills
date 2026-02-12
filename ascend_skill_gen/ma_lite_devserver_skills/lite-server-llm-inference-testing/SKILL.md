---
name: lite-server-llm-inference-testing
description: Guide for automated LLM inference deployment and testing (performance, reliability, functionality, accuracy) on ModelArts Lite Server with Ascend NPU. Covers vLLM, MindIE frameworks and models like LLaMA, Qwen, ChatGLM, DeepSeek. This skill should be used when users need to deploy and test LLM inference services on Lite Server.
license: Apache 2.0
---

# Lite Server LLM Inference Deployment & Testing Guide

## Overview

End-to-end guidance for deploying and testing LLM inference services on ModelArts Lite Server with Ascend NPU. Covers deployment via vLLM / MindIE, and systematic testing for performance, reliability, functionality, and accuracy.

---

## When to Use

- Deploying LLM inference (LLaMA, Qwen, ChatGLM, DeepSeek, Baichuan) on Ascend NPU
- Running LLM performance benchmarks (throughput, latency, TTFT, TPOT)
- Running LLM reliability tests (long-duration, fault recovery, OOM)
- Running LLM functionality tests (API conformance, streaming, tool calling)
- Running LLM accuracy tests (GPU vs NPU output comparison)

---

## Environment Preparation

```bash
# Verify NPU status
npu-smi info

# Verify CANN
cat /usr/local/Ascend/ascend-toolkit/latest/version.cfg

# Verify torch_npu
python3 -c "import torch; import torch_npu; print(torch.npu.is_available(), torch.npu.device_count())"
```

### Model & NPU Planning

| Model Scale | Example | Min NPU (FP16) | Min NPU (INT8/W4) |
|---|---|---|---|
| 7B | Qwen2-7B, LLaMA3-8B | 1× 910B | 1× 910B |
| 13–14B | Qwen1.5-14B, Baichuan2-13B | 2× 910B | 1× 910B |
| 32–34B | Qwen2.5-32B, CodeLlama-34B | 4× 910B | 2× 910B |
| 65–72B | Qwen2-72B, LLaMA2-70B | 8× 910B | 4× 910B |

Reference: [Lite Server算力资源和镜像版本配套关系](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0004.html)

---

## Deployment

### vLLM on Ascend NPU

```bash
# Install vLLM with Ascend backend
pip install vllm-ascend

# Launch OpenAI-compatible API server
python -m vllm.entrypoints.openai.api_server \
  --model /data/models/Qwen2-7B-Chat \
  --tensor-parallel-size 4 \
  --device npu \
  --host 0.0.0.0 --port 8000 \
  --max-model-len 4096 \
  --trust-remote-code
```

### MindIE on Ascend NPU

```bash
# MindIE (Huawei native LLM inference engine)
mindie-service \
  --model /data/models/ChatGLM3-6B \
  --device-id 0,1,2,3 \
  --port 8080 \
  --max-batch-size 32 \
  --max-seq-len 4096
```

### Health Check

```bash
# Verify service is running
curl -s http://localhost:8000/health

# List available models
curl -s http://localhost:8000/v1/models | python3 -m json.tool

# Quick smoke test
curl -s http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"Qwen2-7B-Chat","messages":[{"role":"user","content":"Hello"}],"max_tokens":32}'
```

---

## Performance Testing

### Key Metrics

| Metric | Description | Target |
|---|---|---|
| TTFT | Time To First Token (首 token 延迟) | < 500ms (7B), < 2s (72B) |
| TPOT | Time Per Output Token (逐 token 延迟) | < 50ms |
| Throughput | Tokens per second (output) | Model-dependent |
| QPS | Queries per second under concurrency | Concurrency-dependent |
| HBM Usage | NPU memory consumption | < 90% HBM capacity |

### Benchmark with vLLM Built-in Tool

```bash
# Fixed input/output length benchmark
python -m vllm.entrypoints.openai.bench_serving \
  --base-url http://localhost:8000 \
  --model Qwen2-7B-Chat \
  --num-prompts 200 \
  --request-rate 10 \
  --input-len 512 --output-len 256
```

### Custom Benchmark Script

```python
#!/usr/bin/env python3
"""llm_perf_bench.py — LLM inference performance benchmark"""
import time, json, requests, concurrent.futures, statistics

API_URL = "http://localhost:8000/v1/chat/completions"
MODEL = "Qwen2-7B-Chat"

def single_request(prompt, max_tokens=256):
    payload = {
        "model": MODEL,
        "messages": [{"role": "user", "content": prompt}],
        "max_tokens": max_tokens, "stream": True
    }
    t0 = time.perf_counter()
    ttft = None
    tokens = 0
    with requests.post(API_URL, json=payload, stream=True) as r:
        for line in r.iter_lines():
            if line and line.startswith(b"data: "):
                chunk = line[6:]
                if chunk == b"[DONE]":
                    break
                data = json.loads(chunk)
                if data["choices"][0]["delta"].get("content"):
                    if ttft is None:
                        ttft = time.perf_counter() - t0
                    tokens += 1
    total = time.perf_counter() - t0
    tpot = (total - ttft) / max(tokens - 1, 1) if ttft else 0
    return {"ttft": ttft, "tpot": tpot, "tokens": tokens, "total": total}

def run_benchmark(concurrency=8, num_requests=100):
    prompt = "Explain quantum computing in detail." * 5
    results = []
    with concurrent.futures.ThreadPoolExecutor(max_workers=concurrency) as pool:
        futures = [pool.submit(single_request, prompt) for _ in range(num_requests)]
        for f in concurrent.futures.as_completed(futures):
            results.append(f.result())
    ttfts = [r["ttft"] for r in results if r["ttft"]]
    tpots = [r["tpot"] for r in results if r["tpot"]]
    total_tokens = sum(r["tokens"] for r in results)
    wall_time = max(r["total"] for r in results)
    print(f"Concurrency: {concurrency}")
    print(f"TTFT  — P50: {statistics.median(ttfts)*1000:.0f}ms  P99: {sorted(ttfts)[int(len(ttfts)*0.99)]*1000:.0f}ms")
    print(f"TPOT  — P50: {statistics.median(tpots)*1000:.1f}ms  P99: {sorted(tpots)[int(len(tpots)*0.99)]*1000:.1f}ms")
    print(f"Throughput: {total_tokens/wall_time:.1f} tokens/s")

if __name__ == "__main__":
    run_benchmark()
```

---

## Reliability Testing

### Long-Duration Stability Test

```bash
#!/bin/bash
# llm_stability_test.sh — Run continuous requests for N hours
DURATION_HOURS=${1:-24}
API_URL="http://localhost:8000/v1/chat/completions"
MODEL="Qwen2-7B-Chat"
LOG_FILE="/var/log/llm_stability_$(date +%Y%m%d_%H%M%S).log"
END_TIME=$(($(date +%s) + DURATION_HOURS * 3600))
SUCCESS=0; FAIL=0; ROUND=0

echo "Stability test: ${DURATION_HOURS}h starting at $(date)" | tee "$LOG_FILE"
while [ $(date +%s) -lt $END_TIME ]; do
  ROUND=$((ROUND+1))
  HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" "$API_URL" \
    -H "Content-Type: application/json" \
    -d "{\"model\":\"$MODEL\",\"messages\":[{\"role\":\"user\",\"content\":\"Round $ROUND: Summarize AI.\"}],\"max_tokens\":128}")
  if [ "$HTTP_CODE" = "200" ]; then
    SUCCESS=$((SUCCESS+1))
  else
    FAIL=$((FAIL+1))
    echo "[$(date)] FAIL round=$ROUND http=$HTTP_CODE" >> "$LOG_FILE"
  fi
  sleep 2
done
echo "Done: success=$SUCCESS fail=$FAIL total=$ROUND" | tee -a "$LOG_FILE"
```

### Fault Recovery Test

| Test Case | Method | Expected |
|---|---|---|
| Kill vLLM process | `kill -9 <pid>`, restart, send request | Service recovers, no data corruption |
| NPU device error | Simulate via `npu-smi set -t error-inject` | Service detects error, logs warning |
| OOM trigger | Send max_tokens=32768 on small HBM | Returns 400/500, service stays alive |
| Network interruption | `iptables -A INPUT -p tcp --dport 8000 -j DROP` then restore | Pending requests timeout, new requests succeed |

---

## Functionality Testing

### API Conformance Checklist

| Test Case | Endpoint | Validation |
|---|---|---|
| Chat completion | `POST /v1/chat/completions` | Returns valid JSON with choices |
| Streaming | `stream: true` | Returns SSE chunks, ends with `[DONE]` |
| Multi-turn | Multiple messages in array | Context-aware response |
| System prompt | `role: system` message | Follows system instructions |
| Stop sequences | `stop: ["\n\n"]` | Generation stops at delimiter |
| Temperature | `temperature: 0` | Deterministic output |
| Max tokens | `max_tokens: 10` | Output ≤ 10 tokens |
| Concurrent requests | 50 parallel requests | All return 200, no hang |

### Streaming Validation Script

```python
#!/usr/bin/env python3
"""llm_func_test.py — LLM functionality test suite"""
import requests, json, sys

API = "http://localhost:8000/v1/chat/completions"
MODEL = "Qwen2-7B-Chat"
PASS = 0; FAIL = 0

def test(name, payload, check_fn):
    global PASS, FAIL
    try:
        r = requests.post(API, json=payload, timeout=60)
        assert check_fn(r), f"Check failed: {r.status_code}"
        PASS += 1; print(f"  PASS: {name}")
    except Exception as e:
        FAIL += 1; print(f"  FAIL: {name} — {e}")

def base_payload(**kw):
    p = {"model": MODEL, "messages": [{"role":"user","content":"Say hello"}], "max_tokens": 32}
    p.update(kw)
    return p

print("=== LLM Functionality Tests ===")
test("basic_chat", base_payload(), lambda r: r.status_code==200 and r.json()["choices"])
test("max_tokens", base_payload(max_tokens=5), lambda r: len(r.json()["choices"][0]["message"]["content"].split()) <= 20)
test("temperature_0", base_payload(temperature=0), lambda r: r.status_code==200)
test("stop_sequence", base_payload(stop=["."]), lambda r: not r.json()["choices"][0]["message"]["content"].rstrip().endswith(".."))

print(f"\nResults: {PASS} passed, {FAIL} failed")
sys.exit(1 if FAIL else 0)
```

---

## Accuracy Testing

### GPU vs NPU Output Comparison

```python
#!/usr/bin/env python3
"""llm_accuracy_test.py — Compare NPU inference output against GPU baseline"""
import json, requests, hashlib

GPU_API = "http://<gpu-server>:8000/v1/chat/completions"
NPU_API = "http://localhost:8000/v1/chat/completions"
MODEL = "Qwen2-7B-Chat"

TEST_PROMPTS = [
    "What is the capital of France?",
    "Translate 'hello world' to Chinese.",
    "Write a Python function to compute fibonacci.",
    "Explain the theory of relativity in 3 sentences.",
    "Summarize the benefits of renewable energy.",
]

def query(api, prompt):
    r = requests.post(api, json={
        "model": MODEL, "messages": [{"role":"user","content": prompt}],
        "max_tokens": 256, "temperature": 0
    }, timeout=120)
    return r.json()["choices"][0]["message"]["content"]

print("=== LLM Accuracy: GPU vs NPU ===")
for p in TEST_PROMPTS:
    gpu_out = query(GPU_API, p)
    npu_out = query(NPU_API, p)
    match = gpu_out.strip() == npu_out.strip()
    print(f"Prompt: {p[:50]}...")
    print(f"  Match: {'EXACT' if match else 'DIFF'}")
    if not match:
        print(f"  GPU: {gpu_out[:100]}...")
        print(f"  NPU: {npu_out[:100]}...")
```

### Perplexity Comparison (Offline)

```bash
# Export logprobs from both GPU and NPU, then compare
# vLLM supports logprobs parameter
curl -s http://localhost:8000/v1/completions \
  -d '{"model":"Qwen2-7B-Chat","prompt":"The meaning of life is","max_tokens":64,"logprobs":5,"temperature":0}' \
  | python3 -c "import sys,json; d=json.load(sys.stdin); print('Avg logprob:', sum(d['choices'][0]['logprobs']['token_logprobs'])/len(d['choices'][0]['logprobs']['token_logprobs']))"
```

---

## NPU Monitoring During Tests

```bash
# Real-time NPU utilization and HBM during test
watch -n 1 npu-smi info

# Log NPU metrics to file
nohup bash -c 'while true; do echo "$(date +%s) $(npu-smi info -t usages)" >> /var/log/npu_metrics.log; sleep 5; done' &
```

---

## Reference Documentation

- [LLM/AIGC等模型基于Lite Server适配NPU的训练推理指导](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0014.html)
- [NPU服务器上配置Lite Server资源软件环境](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0011.html)
- [Lite Server算力资源和镜像版本配套关系](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0004.html)
- [Lite Server节点一键式压测](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0035.html)
- [Lite Server节点故障诊断](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0031.html)
