---
name: flask-blueprint-api
description: Use this skill when organizing multiple API domains with Flask Blueprints, setting up separate URL prefixes and CORS policies per API group, or structuring console/service/web/internal APIs. Triggers on keywords like "Flask Blueprint", "API routing", "CORS policy", "multi-API", "URL prefix".
version: 1.0.0
---

# Flask Blueprint 多 API 域路由组织

## Overview

通过 Flask Blueprint 实现多套 API 域的路由组织，每套独立 URL 前缀、CORS 策略和认证方式。

**Keywords**: Flask, Blueprint, API routing, CORS, URL prefix, multi-API

## Dify 源码引用

- `api/extensions/ext_blueprints.py` — 统一注册 5 套 Blueprint
- `api/controllers/console/` — 管理后台 API（JWT 认证）
- `api/controllers/service_api/` — 对外服务 API（API Key 认证）
- `api/controllers/web/` — 终端用户 API
- `api/controllers/inner_api/` — 内部微服务 API（无认证/内网隔离）

## Blueprint 注册骨架

```python
BLUEPRINTS = [
    {"name": "console", "url_prefix": "/console/api", "auth": "jwt",
     "cors_origins": ["https://admin.example.com"]},
    {"name": "service_api", "url_prefix": "/v1", "auth": "api_key",
     "cors_origins": "*"},
    {"name": "web", "url_prefix": "/api", "auth": "end_user_token",
     "cors_origins": "*"},
    {"name": "inner_api", "url_prefix": "/inner/api", "auth": "none",
     "cors_origins": []},
]

def init_app(app: Flask):
    for bp_config in BLUEPRINTS:
        bp = import_module(f"controllers.{bp_config['name']}").bp
        app.register_blueprint(bp, url_prefix=bp_config["url_prefix"])
        CORS(bp, origins=bp_config["cors_origins"])
```

## 关键规则

- 每套 Blueprint 独立 URL 前缀、CORS 策略和认证方式
- Controller 文件按 Blueprint 目录隔离，禁止跨 Blueprint 导入
- 公开 API（service_api）必须有限流和 API Key 校验
- 内部 API 通过网络隔离保护，不暴露到公网

## 反模式

- 所有路由注册在同一个 Blueprint 下，无法差异化 CORS
- 管理接口和公开接口共用同一认证策略
