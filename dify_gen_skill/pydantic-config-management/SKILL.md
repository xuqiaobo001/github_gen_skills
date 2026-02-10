---
name: pydantic-config-management
description: Use this skill when implementing type-safe configuration management with Pydantic BaseSettings, organizing config by functional domain using Mixins, or setting up environment-variable-driven configuration. Triggers on keywords like "Pydantic config", "BaseSettings", "environment variables", "config management", "frozen config".
version: 1.0.0
license: Apache 2.0, see LICENSE.txt
---

# Pydantic 配置管理体系

## Overview

基于 Pydantic BaseSettings 实现类型安全、环境变量驱动的配置管理，按功能域 Mixin 组织。

**Keywords**: Pydantic, BaseSettings, config, environment variables, Mixin, frozen config

## Dify 源码引用

- `api/configs/app_config.py` — 主配置类，继承多个 Mixin
- `api/configs/feature/` — 按功能域分组的配置 Mixin
- `api/configs/remote_settings_sources/` — Apollo/Nacos 远程配置加载

## 核心代码骨架

```python
class DatabaseConfig(BaseSettings):
    DB_HOST: str = "localhost"
    DB_PORT: int = 5432
    DB_USERNAME: str = "postgres"
    DB_PASSWORD: str = ""
    SQLALCHEMY_POOL_SIZE: int = 30

class RedisConfig(BaseSettings):
    REDIS_HOST: str = "localhost"
    REDIS_PORT: int = 6379
    REDIS_PASSWORD: str = ""

class DifyConfig(DatabaseConfig, RedisConfig, FeatureFlagsConfig):
    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        frozen=True,
    )

dify_config = DifyConfig()
```

## 关键规则

- 每个配置项必须有类型注解和默认值
- 按功能域拆分 Mixin，主类通过多继承组合
- 使用 `frozen=True` 防止运行时意外修改配置
- 敏感配置通过环境变量注入，不写入代码

## 反模式

- 所有配置项堆在一个类中（超过 200 行难以维护）
- 配置项没有默认值，部署时缺少环境变量直接崩溃
- 在业务代码中 `os.getenv()` 读取配置，绕过类型校验
