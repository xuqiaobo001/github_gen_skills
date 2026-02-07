# Skill 6: `pydantic-config-management` — Pydantic 配置管理体系

**适用场景**: 任何需要类型安全、环境变量驱动、支持远程配置的 Python 项目。

**Dify 源码引用**:

- `api/configs/app_config.py` — 主配置类，继承多个 Mixin
- `api/configs/feature/` — 按功能域分组的配置 Mixin
- `api/configs/remote_settings_sources/` — Apollo/Nacos 远程配置加载

**核心代码骨架**:

```python
# 1. 按功能域定义配置 Mixin
class DatabaseConfig(BaseSettings):
    DB_HOST: str = "localhost"
    DB_PORT: int = 5432
    DB_USERNAME: str = "postgres"
    DB_PASSWORD: str = ""
    DB_DATABASE: str = "dify"
    SQLALCHEMY_POOL_SIZE: int = 30

class RedisConfig(BaseSettings):
    REDIS_HOST: str = "localhost"
    REDIS_PORT: int = 6379
    REDIS_PASSWORD: str = ""
    REDIS_USE_SSL: bool = False

class FeatureFlagsConfig(BaseSettings):
    ENABLE_BILLING: bool = False
    ENABLE_EMAIL_VERIFICATION: bool = False

# 2. 主配置类组合所有 Mixin
class DifyConfig(DatabaseConfig, RedisConfig, FeatureFlagsConfig):
    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        frozen=True,  # 不可变
    )

# 3. 全局单例访问
dify_config = DifyConfig()
```

**关键规则**:

- 每个配置项必须有类型注解和默认值
- 按功能域拆分 Mixin，主类通过多继承组合
- 使用 `frozen=True` 防止运行时意外修改配置
- 敏感配置（密码、密钥）通过环境变量注入，不写入代码

**反模式**:

- ❌ 所有配置项堆在一个类中（超过 200 行难以维护）
- ❌ 配置项没有默认值，部署时缺少环境变量直接崩溃
- ❌ 在业务代码中 `os.getenv()` 读取配置，绕过类型校验
