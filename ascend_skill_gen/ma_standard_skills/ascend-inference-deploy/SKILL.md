---
name: ascend-inference-deploy
description: Guide for deploying AI models as inference services on Huawei Ascend NPU via ModelArts Standard. This skill should be used when users need to deploy models for real-time or batch inference on Ascend NPU, configure online service endpoints, optimize inference performance, or set up access authentication.
license: Apache 2.0
---

# Ascend Inference Deployment Guide

## Overview

This skill provides guidance for deploying trained AI models as inference services on Huawei Ascend NPU hardware within ModelArts Standard. It covers model packaging, real-time and batch inference deployment, access channel configuration, and performance optimization.

---

## When to Use

- Deploying a trained model as a real-time inference service on Ascend NPU
- Deploying batch inference jobs on Ascend NPU
- Writing model configuration files (config.json) for ModelArts
- Writing custom inference code (customize_service.py)
- Configuring service access channels (public, VPC, VPC high-speed)
- Optimizing inference latency and throughput on NPU

---

## Model Packaging

### Model Package Structure

ModelArts requires a specific directory structure for model deployment:

```
model/
├── model/
│   ├── config.json              # Model configuration (required)
│   ├── customize_service.py     # Custom inference code (required for custom logic)
│   └── model_weights/           # Model weight files
│       ├── pytorch_model.bin
│       └── config.json
├── requirements.txt             # Python dependencies (optional)
└── README.md                    # Model description (optional)
```

### config.json Specification

```json
{
    "model_algorithm": "custom_model",
    "model_type": "PyTorch",
    "runtime": "pytorch_2.1.0-cann_8.0-py_3.9",
    "apis": [
        {
            "url": "/predict",
            "method": "POST",
            "request": {
                "Content-type": "application/json"
            },
            "response": {
                "Content-type": "application/json"
            }
        }
    ]
}
```

Reference: [模型配置文件编写说明](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/inference-modelarts-0056.html)

---

## Custom Inference Code

### Basic Inference Service Template

```python
import torch
import torch_npu
from model_service.pytorch_model_service import PTServingBaseService

class MyInferenceService(PTServingBaseService):
    def __init__(self, model_name, model_path):
        super().__init__(model_name, model_path)
        self.device = torch.device("npu:0")
        self.model = self._load_model(model_path)

    def _load_model(self, model_path):
        model = torch.load(model_path, map_location=self.device)
        model.eval()
        return model

    def _preprocess(self, data):
        """Preprocess input data."""
        input_data = data.get("input", None)
        # Transform input to tensor
        tensor = torch.tensor(input_data).to(self.device)
        return tensor

    def _inference(self, preprocessed_data):
        """Run inference on NPU."""
        with torch.no_grad():
            result = self.model(preprocessed_data)
        return result

    def _postprocess(self, inference_result):
        """Postprocess model output."""
        return {"result": inference_result.cpu().tolist()}
```

Reference: [模型推理代码编写说明](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/inference-modelarts-0057.html)

---

## Deployment Modes

### Real-time Inference Service

Deploy model as an always-on HTTP service:

- **Use case**: Low-latency predictions, interactive applications
- **Protocols**: HTTP/HTTPS, WebSocket, Server-Sent Events (SSE)
- **Scaling**: Auto-scaling based on request volume

Key configuration parameters:
- **Instance count**: Min/max instances for auto-scaling
- **Resource flavor**: Ascend NPU specification
- **Timeout**: Request timeout in seconds

Reference: [部署模型为在线服务](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/inference-modelarts-0018.html)

### Batch Inference Service

Deploy model for processing large datasets:

- **Use case**: Offline scoring, bulk predictions
- **Input**: Data from OBS
- **Output**: Results written to OBS

Reference: [将模型部署为批量推理服务](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/inference-modelarts-0039.html)

---

## Access Configuration

### Authentication Methods

| Method | Use Case | Reference |
|---|---|---|
| Token | Temporary access, testing | [Token认证](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/inference-modelarts-0023.html) |
| AK/SK | Programmatic access, production | [AK/SK认证](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/inference-modelarts-0024.html) |
| APP | Third-party application integration | [APP认证](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/inference-modelarts-0025.html) |

### Access Channels

| Channel | Latency | Security | Use Case |
|---|---|---|---|
| Public network | Higher | TLS required | External clients |
| VPC | Lower | VPC isolation | Internal services |
| VPC high-speed | Lowest | VPC isolation | High-throughput |

---

## Performance Optimization

### NPU Inference Optimization Checklist

1. **Model compilation**: Use `torch.compile()` or `torch_npu` graph mode
2. **Batch processing**: Aggregate requests for batch inference
3. **Mixed precision**: Use FP16 or INT8 quantization for inference
4. **Memory management**: Pre-allocate NPU memory, avoid dynamic allocation

### Streaming Inference (SSE/WebSocket)

For LLM token-by-token generation:

```python
# SSE endpoint for streaming inference
@app.route("/predict/stream", methods=["POST"])
def stream_predict():
    def generate():
        for token in model.generate_stream(input_data):
            yield f"data: {json.dumps({'token': token})}\n\n"
    return Response(generate(), mimetype="text/event-stream")
```

Reference: [使用Server-Sent Events协议的方式访问在线服务](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/inference-modelarts-0124.html)

---

## Reference Documentation

- [推理部署使用场景](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/inference-modelarts-0001.html)
- [模型包结构介绍](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/inference-modelarts-0055.html)
- [自定义引擎创建模型规范](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/inference-modelarts-0128.html)
- [设置在线服务故障自动重启](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/inference-modelarts-0135.html)
