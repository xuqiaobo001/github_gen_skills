---
name: lite-server-audio-mm-inference-testing
description: Guide for automated audio and multimodal model inference deployment and testing (performance, reliability, functionality, accuracy) on ModelArts Lite Server with Ascend NPU. Covers ASR (Whisper, Paraformer), TTS (CosyVoice, VITS), and multimodal models (LLaVA, Qwen-VL, InternVL). This skill should be used when users need to deploy and test audio or multimodal inference on Lite Server.
license: Apache 2.0
---

# Lite Server Audio & Multimodal Inference Deployment & Testing Guide

## Overview

End-to-end guidance for deploying and testing audio (ASR/TTS) and multimodal (vision-language) inference services on ModelArts Lite Server with Ascend NPU. Covers systematic testing for performance, reliability, functionality, and accuracy.

---

## When to Use

- Deploying ASR models (Whisper, Paraformer, SenseVoice) on Ascend NPU
- Deploying TTS models (CosyVoice, VITS, ChatTTS) on Ascend NPU
- Deploying multimodal models (LLaVA, Qwen-VL, InternVL, CogVLM) on NPU
- Running audio/multimodal performance benchmarks
- Running reliability tests (long-duration, concurrent streams)
- Running functionality tests (multi-language, multi-image, streaming)
- Running accuracy tests (WER/CER for ASR, similarity for multimodal)

---

## Model & NPU Planning

### Audio Models

| Model | Task | Min NPU | HBM Estimate |
|---|---|---|---|
| Whisper-large-v3 | ASR | 1× 910B | ~4 GB |
| Paraformer-large | ASR (streaming) | 1× 910B | ~2 GB |
| SenseVoice-small | ASR (multilingual) | 1× 910B | ~2 GB |
| CosyVoice | TTS | 1× 910B | ~6 GB |
| VITS / ChatTTS | TTS | 1× 910B | ~3 GB |

### Multimodal Models

| Model | Task | Min NPU (FP16) | HBM Estimate |
|---|---|---|---|
| LLaVA-1.5-7B | Vision-Language | 1× 910B | ~16 GB |
| Qwen-VL-Chat | Vision-Language | 2× 910B | ~20 GB |
| InternVL2-8B | Vision-Language | 1× 910B | ~18 GB |
| CogVLM2-19B | Vision-Language | 4× 910B | ~40 GB |
| Qwen2-Audio-7B | Audio-Language | 1× 910B | ~16 GB |

---

## Audio Model Deployment

### ASR: Whisper on Ascend NPU

```python
#!/usr/bin/env python3
"""asr_serve.py — Whisper ASR inference server on NPU"""
import torch, torch_npu, whisper, tempfile, os
from flask import Flask, request, jsonify

app = Flask(__name__)
model = whisper.load_model("large-v3", device="npu:0")

@app.route("/transcribe", methods=["POST"])
def transcribe():
    audio_file = request.files.get("audio")
    language = request.form.get("language", None)
    tmp = tempfile.NamedTemporaryFile(suffix=".wav", delete=False)
    audio_file.save(tmp.name)
    result = model.transcribe(tmp.name, language=language)
    os.unlink(tmp.name)
    return jsonify({"text": result["text"], "language": result["language"],
                     "segments": [{"start": s["start"], "end": s["end"],
                                   "text": s["text"]} for s in result["segments"]]})

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8001)
```

### TTS: CosyVoice on Ascend NPU

```python
#!/usr/bin/env python3
"""tts_serve.py — CosyVoice TTS inference server on NPU"""
import torch, torch_npu, io
from flask import Flask, request, send_file
from cosyvoice import CosyVoice

app = Flask(__name__)
tts = CosyVoice("/data/models/CosyVoice-300M", device="npu:0")

@app.route("/synthesize", methods=["POST"])
def synthesize():
    data = request.json
    text = data["text"]
    speaker = data.get("speaker", "default")
    audio = tts.inference_sft(text, speaker)
    buf = io.BytesIO()
    import torchaudio
    torchaudio.save(buf, audio["tts_speech"], 22050, format="wav")
    buf.seek(0)
    return send_file(buf, mimetype="audio/wav")

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8002)
```

---

## Multimodal Model Deployment

### LLaVA / Qwen-VL via vLLM

```bash
# vLLM supports multimodal models with --image-input
python -m vllm.entrypoints.openai.api_server \
  --model /data/models/Qwen2-VL-7B-Instruct \
  --tensor-parallel-size 2 \
  --device npu \
  --host 0.0.0.0 --port 8003 \
  --trust-remote-code
```

### Multimodal API Call Example

```bash
# Send image + text query
curl -s http://localhost:8003/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Qwen2-VL-7B-Instruct",
    "messages": [{"role":"user","content":[
      {"type":"image_url","image_url":{"url":"data:image/png;base64,<BASE64>"}},
      {"type":"text","text":"Describe this image in detail."}
    ]}],
    "max_tokens": 256
  }'
```

---

## Performance Testing

### Audio Metrics

| Metric | Description | Target (Whisper-large-v3) |
|---|---|---|
| RTF | Real-Time Factor (processing time / audio duration) | < 0.3 |
| Latency | Per-utterance inference time | < 2s for 10s audio |
| Throughput | Audio seconds processed per wall second | > 3× real-time |
| TTS Latency | Time to generate 1s of speech | < 1s |

### Multimodal Metrics

| Metric | Description | Target (7B model) |
|---|---|---|
| TTFT | Time To First Token (with image) | < 2s |
| TPOT | Time Per Output Token | < 60ms |
| Image Encode | Image preprocessing + encoding time | < 500ms |
| Throughput | Tokens/s under concurrency | Model-dependent |

### Audio Benchmark Script

```python
#!/usr/bin/env python3
"""audio_perf_bench.py — ASR/TTS performance benchmark"""
import time, requests, statistics, os, glob

ASR_API = "http://localhost:8001/transcribe"
TTS_API = "http://localhost:8002/synthesize"

def bench_asr(audio_dir="/data/test_audio", num_files=50):
    files = glob.glob(f"{audio_dir}/*.wav")[:num_files]
    latencies = []; audio_durations = []
    for f in files:
        duration = os.path.getsize(f) / (16000 * 2)  # rough estimate: 16kHz 16bit mono
        t0 = time.perf_counter()
        r = requests.post(ASR_API, files={"audio": open(f,"rb")}, timeout=120)
        lat = time.perf_counter() - t0
        if r.status_code == 200:
            latencies.append(lat); audio_durations.append(duration)
    total_audio = sum(audio_durations)
    total_proc = sum(latencies)
    print(f"=== ASR Benchmark ===")
    print(f"Files: {len(latencies)}, Total audio: {total_audio:.1f}s")
    print(f"RTF: {total_proc/total_audio:.3f}")
    print(f"Latency P50: {statistics.median(latencies)*1000:.0f}ms")

def bench_tts(num_requests=30):
    texts = ["今天天气真不错，适合出去散步。", "The quick brown fox jumps over the lazy dog.",
             "人工智能正在改变世界的方方面面。"] * 10
    latencies = []
    for t in texts[:num_requests]:
        t0 = time.perf_counter()
        r = requests.post(TTS_API, json={"text": t}, timeout=60)
        lat = time.perf_counter() - t0
        if r.status_code == 200:
            latencies.append(lat)
    print(f"=== TTS Benchmark ===")
    print(f"Requests: {len(latencies)}")
    print(f"Latency P50: {statistics.median(latencies)*1000:.0f}ms  P99: {sorted(latencies)[int(len(latencies)*0.99)]*1000:.0f}ms")

if __name__ == "__main__":
    bench_asr()
    bench_tts()
```

---

## Reliability Testing

### Long-Duration Stability

```bash
#!/bin/bash
# audio_mm_stability_test.sh — Continuous audio/multimodal requests for N hours
DURATION_HOURS=${1:-12}
ASR_API="http://localhost:8001/transcribe"
MM_API="http://localhost:8003/v1/chat/completions"
LOG="/var/log/audio_mm_stability_$(date +%Y%m%d_%H%M%S).log"
END=$(($(date +%s) + DURATION_HOURS * 3600))
ASR_OK=0; ASR_NG=0; MM_OK=0; MM_NG=0; ROUND=0

echo "Audio+MM stability test: ${DURATION_HOURS}h" | tee "$LOG"
while [ $(date +%s) -lt $END ]; do
  ROUND=$((ROUND+1))
  # ASR test — generate 3s silence wav as dummy
  python3 -c "import wave,struct;f=wave.open('/tmp/test.wav','w');f.setnchannels(1);f.setsampwidth(2);f.setframerate(16000);f.writeframes(struct.pack('<'+'h'*48000,*([0]*48000)));f.close()"
  CODE=$(curl -s -o /dev/null -w "%{http_code}" "$ASR_API" -F "audio=@/tmp/test.wav" --max-time 30)
  if [ "$CODE" = "200" ]; then ASR_OK=$((ASR_OK+1)); else ASR_NG=$((ASR_NG+1)); echo "[$(date)] ASR FAIL round=$ROUND http=$CODE" >> "$LOG"; fi

  # Multimodal test
  CODE=$(curl -s -o /dev/null -w "%{http_code}" "$MM_API" \
    -H "Content-Type: application/json" \
    -d '{"model":"Qwen2-VL-7B-Instruct","messages":[{"role":"user","content":"Describe AI."}],"max_tokens":32}' --max-time 30)
  if [ "$CODE" = "200" ]; then MM_OK=$((MM_OK+1)); else MM_NG=$((MM_NG+1)); echo "[$(date)] MM FAIL round=$ROUND http=$CODE" >> "$LOG"; fi
  sleep 3
done
echo "ASR: ok=$ASR_OK fail=$ASR_NG | MM: ok=$MM_OK fail=$MM_NG | rounds=$ROUND" | tee -a "$LOG"
```

### Fault Scenarios

| Test Case | Target | Method | Expected |
|---|---|---|---|
| Corrupted audio | ASR | Send random bytes as .wav | Returns error, no crash |
| Empty audio | ASR | Send 0-length wav | Returns empty text or error |
| Oversized audio | ASR | Send 60min audio file | Timeout or OOM handled gracefully |
| Invalid image | Multimodal | Send corrupted base64 | Returns error, service alive |
| No image input | Multimodal | Text-only to VL model | Falls back to text-only response |
| Concurrent streams | ASR+TTS | 20 parallel requests | All complete, no deadlock |

---

## Functionality Testing

### Audio Functionality

| Test Case | Input | Validation |
|---|---|---|
| Chinese ASR | Chinese speech wav | Correct Chinese transcription |
| English ASR | English speech wav | Correct English transcription |
| Multi-language | Mixed language audio | Correct language detection + transcription |
| Timestamp segments | Any audio | Segments with start/end timestamps |
| Long audio | 30min+ audio | Complete transcription, no truncation |
| TTS basic | Chinese text | Valid wav output, audible speech |
| TTS speaker | text + speaker_id | Voice matches specified speaker |
| TTS streaming | text + stream=true | Chunked audio output |

### Multimodal Functionality

| Test Case | Input | Validation |
|---|---|---|
| Single image + text | 1 image + question | Relevant description |
| Multi-image | 2+ images + comparison query | References both images |
| Image resolution | 4K image input | Handles without OOM |
| OCR capability | Image with text | Extracts text correctly |
| Chart understanding | Chart/graph image | Describes data trends |
| Video frames | Multiple frames + query | Temporal understanding |
| Streaming output | stream: true | SSE chunks returned |

---

## Accuracy Testing

### ASR: WER/CER Comparison (GPU vs NPU)

```python
#!/usr/bin/env python3
"""asr_accuracy_test.py — Compare ASR accuracy between GPU and NPU"""
import requests, json

GPU_ASR = "http://<gpu-server>:8001/transcribe"
NPU_ASR = "http://localhost:8001/transcribe"

# Test dataset: list of (audio_path, ground_truth_text)
TEST_SET = [
    ("/data/test_audio/zh_001.wav", "今天的天气非常好"),
    ("/data/test_audio/zh_002.wav", "人工智能正在改变世界"),
    ("/data/test_audio/en_001.wav", "the quick brown fox jumps over the lazy dog"),
    ("/data/test_audio/en_002.wav", "artificial intelligence is transforming industries"),
]

def transcribe(api, audio_path):
    r = requests.post(api, files={"audio": open(audio_path, "rb")}, timeout=120)
    return r.json()["text"].strip()

def cer(ref, hyp):
    """Character Error Rate"""
    import editdistance
    return editdistance.eval(ref, hyp) / max(len(ref), 1)

def wer(ref, hyp):
    """Word Error Rate"""
    import editdistance
    ref_words, hyp_words = ref.split(), hyp.split()
    return editdistance.eval(ref_words, hyp_words) / max(len(ref_words), 1)

print("=== ASR Accuracy: GPU vs NPU ===")
for audio, gt in TEST_SET:
    gpu_text = transcribe(GPU_ASR, audio)
    npu_text = transcribe(NPU_ASR, audio)
    gpu_cer = cer(gt, gpu_text)
    npu_cer = cer(gt, npu_text)
    print(f"Audio: {audio}")
    print(f"  GT:  {gt}")
    print(f"  GPU: {gpu_text}  CER={gpu_cer:.3f}")
    print(f"  NPU: {npu_text}  CER={npu_cer:.3f}")
    print(f"  Diff: {abs(gpu_cer - npu_cer):.3f}")
```

> CER 差异 < 1% 视为精度对齐通过。

### Multimodal: GPU vs NPU Output Comparison

```python
#!/usr/bin/env python3
"""mm_accuracy_test.py — Compare multimodal output between GPU and NPU"""
import requests, json, base64

GPU_API = "http://<gpu-server>:8003/v1/chat/completions"
NPU_API = "http://localhost:8003/v1/chat/completions"
MODEL = "Qwen2-VL-7B-Instruct"

TEST_CASES = [
    {"image": "/data/test_images/cat.jpg", "question": "What animal is in this image?", "expected_keyword": "cat"},
    {"image": "/data/test_images/chart.png", "question": "What trend does this chart show?", "expected_keyword": "increase"},
    {"image": "/data/test_images/street.jpg", "question": "How many people are in this image?", "expected_keyword": None},
]

def encode_image(path):
    with open(path, "rb") as f:
        return base64.b64encode(f.read()).decode()

def query_mm(api, image_path, question):
    b64 = encode_image(image_path)
    r = requests.post(api, json={
        "model": MODEL,
        "messages": [{"role": "user", "content": [
            {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{b64}"}},
            {"type": "text", "text": question}
        ]}],
        "max_tokens": 128, "temperature": 0
    }, timeout=120)
    return r.json()["choices"][0]["message"]["content"]

print("=== Multimodal Accuracy: GPU vs NPU ===")
for tc in TEST_CASES:
    gpu_out = query_mm(GPU_API, tc["image"], tc["question"])
    npu_out = query_mm(NPU_API, tc["image"], tc["question"])
    kw_match = tc["expected_keyword"].lower() in npu_out.lower() if tc["expected_keyword"] else "N/A"
    print(f"Q: {tc['question']}")
    print(f"  GPU: {gpu_out[:80]}...")
    print(f"  NPU: {npu_out[:80]}...")
    print(f"  Keyword match: {kw_match}")
```

---

## NPU Monitoring During Tests

```bash
# Real-time NPU utilization and HBM
watch -n 1 npu-smi info

# Log NPU metrics to file during test
nohup bash -c 'while true; do echo "$(date +%s) $(npu-smi info -t usages)" >> /var/log/npu_metrics.log; sleep 5; done' &
```

---

## Reference Documentation

- [LLM/AIGC等模型基于Lite Server适配NPU的训练推理指导](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0014.html)
- [NPU服务器上配置Lite Server资源软件环境](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0011.html)
- [Lite Server算力资源和镜像版本配套关系](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0004.html)
- [Lite Server节点一键式压测](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0035.html)
- [Lite Server节点故障诊断](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0031.html)
- [配置Lite Server存储](https://support.huaweicloud.com/intl/zh-cn/usermanual-server-modelarts/usermanual-server-0009.html)
