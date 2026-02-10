---
name: plugin-extension-system
description: Use this skill when designing a plugin/extension system, implementing abstract base classes for plugin contracts, using Registry pattern for plugin management, or building Provider Adapter patterns. Triggers on keywords like "plugin system", "extension", "registry pattern", "provider adapter", "plugin architecture".
version: 1.0.0
---

# 插件化扩展体系

## Overview

通过抽象基类定义插件契约、Registry 模式管理插件、Strategy 模式实现工具、Provider Adapter 适配供应商。

**Keywords**: plugin, extension, registry, strategy pattern, provider adapter

## Dify 源码引用

- `api/core/plugin/` — 插件生命周期管理
- `api/core/tools/` — 50+ 内置工具统一接口
- `api/core/model_runtime/` — 多 LLM Provider Adapter 模式

## 核心设计模式

```python
# 1. Strategy 模式
class BuiltinTool(ABC):
    @abstractmethod
    def _invoke(self, user_id: str, tool_parameters: dict) -> ToolInvokeMessage: ...

# 2. Provider Adapter 模式
class ModelProvider(ABC):
    @abstractmethod
    def validate_provider_credentials(self, credentials: dict) -> None: ...
    @abstractmethod
    def get_model_instance(self, model_type: ModelType) -> AIModel: ...

# 3. Registry + Factory
class ModelProviderFactory:
    _providers: dict[str, type[ModelProvider]] = {}

    @classmethod
    def register(cls, name: str, provider_cls: type[ModelProvider]):
        cls._providers[name] = provider_cls

    @classmethod
    def get_provider(cls, name: str) -> ModelProvider:
        if name not in cls._providers:
            raise ProviderNotFoundError(name)
        return cls._providers[name]()
```

## 关键规则

- 插件通过抽象基类定义接口契约
- 使用注册表模式管理插件实例，避免硬编码
- 插件凭证校验在加载时执行，失败则拒绝注册
- 插件间通过消息传递通信，禁止直接互相导入

## 反模式

- 插件直接 `import` 其他插件的内部实现
- 在主代码中 `if provider == "openai"` 硬编码分支
- 插件接口过于宽泛
