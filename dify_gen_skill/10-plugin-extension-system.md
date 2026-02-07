# Skill 10: `plugin-extension-system` — 插件化扩展体系

**适用场景**: 需要支持第三方扩展、动态加载功能模块的平台型产品。

**Dify 源码引用**:

- `api/core/plugin/` — 插件生命周期管理（安装、启用、禁用、卸载）
- `api/core/tools/` — 50+ 内置工具的统一接口（`BuiltinTool` 基类）
- `api/core/model_runtime/` — 多 LLM Provider 的 Adapter 模式

**核心设计模式**:

```python
# 1. 统一工具接口（Strategy 模式）
class BuiltinTool(ABC):
    @abstractmethod
    def _invoke(self, user_id: str, tool_parameters: dict) -> ToolInvokeMessage:
        """每个工具实现自己的调用逻辑"""
        ...

    def validate_credentials(self, credentials: dict) -> None:
        """凭证校验（可选覆盖）"""
        ...

# 2. Provider Adapter 模式
class ModelProvider(ABC):
    @abstractmethod
    def validate_provider_credentials(self, credentials: dict) -> None: ...

    @abstractmethod
    def get_model_instance(self, model_type: ModelType) -> AIModel: ...

# 3. Factory 动态创建
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

**关键规则**:

- 插件通过抽象基类定义接口契约，第三方只需实现接口
- 使用注册表模式（`register` + `get`）管理插件实例，避免硬编码
- 插件凭证校验在加载时执行，失败则拒绝注册
- 插件间通过消息传递通信，禁止直接互相导入

**反模式**:

- ❌ 插件直接 `import` 其他插件的内部实现
- ❌ 在主代码中 `if provider == "openai"` 硬编码分支
- ❌ 插件接口过于宽泛（一个 `execute(params)` 包打天下）
