# Skill 9: `domain-error-hierarchy` — 领域异常层次体系

**适用场景**: 需要清晰错误分类、统一错误响应格式的后端项目。

**Dify 源码引用**:

- `api/core/errors/error.py` — 领域基础异常（`LLMBadRequestError` 等）
- `api/services/errors/` — 服务层异常（`AppNotFoundError` 等）
- `api/libs/external_api.py` — 全局异常处理器，异常→HTTP 状态码映射

**异常层次骨架**:

```python
# 1. 领域层异常（core/errors/）
class DifyBaseError(Exception):
    """所有 Dify 异常的基类"""
    def __init__(self, description: str = ""):
        self.description = description

class LLMError(DifyBaseError):
    """LLM 调用相关异常基类"""

class LLMBadRequestError(LLMError):
    """LLM 请求参数错误"""

class ProviderTokenNotInitError(LLMError):
    """Provider Token 未配置"""

# 2. 服务层异常（services/errors/）
class AppNotFoundError(DifyBaseError):
    """应用不存在"""

class AccountBannedError(DifyBaseError):
    """账户被封禁"""

# 3. 全局异常处理器（libs/external_api.py）
ERROR_STATUS_MAP = {
    LLMBadRequestError: 400,
    AppNotFoundError: 404,
    AccountBannedError: 403,
    LLMError: 500,  # 兜底
}

@app.errorhandler(DifyBaseError)
def handle_dify_error(error):
    status = ERROR_STATUS_MAP.get(type(error), 500)
    return jsonify({"code": type(error).__name__,
                    "message": error.description}), status
```

**关键规则**:

- 每层只抛出本层定义的异常，上层捕获并转换为本层异常
- Controller 层不捕获异常，由全局处理器统一转换为 HTTP 响应
- 异常类名即错误码（`LLMBadRequestError` → `"LLMBadRequestError"`）
- 所有异常必须继承 `DifyBaseError`，便于全局捕获

**反模式**:

- ❌ 在 Controller 中 `try/except Exception` 吞掉所有异常
- ❌ 直接 `raise HTTPException(400)` 绕过领域异常体系
- ❌ 异常类不携带描述信息，前端无法展示有意义的错误提示
