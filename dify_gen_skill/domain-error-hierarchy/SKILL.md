---
name: domain-error-hierarchy
description: Use this skill when designing a hierarchical exception system, implementing global error handlers that map exceptions to HTTP status codes, or organizing domain/service layer errors. Triggers on keywords like "error hierarchy", "exception handling", "domain error", "error handler", "HTTP status mapping".
version: 1.0.0
license: Apache 2.0, see LICENSE.txt
---

# 领域异常层次体系

## Overview

构建层次化异常结构，通过全局异常处理器将领域异常映射为 HTTP 状态码。

**Keywords**: error hierarchy, exception, domain error, error handler, HTTP status code

## Dify 源码引用

- `api/core/errors/error.py` — 领域基础异常
- `api/services/errors/` — 服务层异常
- `api/libs/external_api.py` — 全局异常处理器

## 异常层次骨架

```python
# 1. 领域层异常
class DifyBaseError(Exception):
    def __init__(self, description: str = ""):
        self.description = description

class LLMError(DifyBaseError): ...
class LLMBadRequestError(LLMError): ...

# 2. 服务层异常
class AppNotFoundError(DifyBaseError): ...
class AccountBannedError(DifyBaseError): ...

# 3. 全局异常处理器
ERROR_STATUS_MAP = {
    LLMBadRequestError: 400,
    AppNotFoundError: 404,
    AccountBannedError: 403,
    LLMError: 500,
}

@app.errorhandler(DifyBaseError)
def handle_dify_error(error):
    status = ERROR_STATUS_MAP.get(type(error), 500)
    return jsonify({"code": type(error).__name__,
                    "message": error.description}), status
```

## 关键规则

- 每层只抛出本层定义的异常，上层捕获并转换
- Controller 层不捕获异常，由全局处理器统一转换为 HTTP 响应
- 异常类名即错误码
- 所有异常必须继承 `DifyBaseError`

## 反模式

- 在 Controller 中 `try/except Exception` 吞掉所有异常
- 直接 `raise HTTPException(400)` 绕过领域异常体系
- 异常类不携带描述信息
