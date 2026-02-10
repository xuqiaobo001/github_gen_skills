---
name: jwt-multi-auth
description: Use this skill when implementing JWT authentication, multi-strategy auth (JWT/API Key/OAuth), token extraction from headers/cookies/query params, or CSRF protection. Triggers on keywords like "JWT auth", "multi-auth", "API key authentication", "token verification", "CSRF token".
version: 1.0.0
license: Apache 2.0, see LICENSE.txt
---

# JWT 多策略认证体系

## Overview

实现同时支持 JWT、API Key、OAuth 多种认证方式的后端系统，包含 Token 提取优先级和 CSRF 保护。

**Keywords**: JWT, authentication, API key, OAuth, token, CSRF, multi-auth

## Dify 源码引用

- `api/services/passport_service.py` — JWT 签发/验证（HS256）
- `api/extensions/ext_login.py` — `request_loader` 多策略分发
- `api/services/api_token_service.py` — Service API Token 管理
- `api/libs/passport.py` — Token 提取工具函数

## 认证分发流程

```python
@login_manager.request_loader
def load_user_from_request(request):
    token = extract_access_token(request)
    decoded = PassportService.verify(token)

    user_type = decoded.get("token_type")
    if user_type == "access":
        return AccountService.load_logged_in_account(decoded["user_id"])
    elif user_type == "end_user":
        return EndUser.query.get(decoded["end_user_id"])
    elif user_type == "app":
        return AppMCPServer.query.get(decoded["app_id"])
    return None
```

**Token 提取优先级**: `Authorization: Bearer <token>` → Cookie `access_token` → Query `token`

## 关键规则

- JWT 仅用 HS256 签名，密钥从 `SECRET_KEY` 配置读取
- API Key 认证走独立路径（`ApiTokenService`），不经过 JWT 流程
- CSRF Token 在登录时生成，写入 Cookie，POST 请求时校验
- Token 过期时间通过配置控制，不硬编码

## 反模式

- 在多个地方重复实现 token 提取逻辑
- JWT payload 中存储敏感信息（密码、密钥）
- 忘记区分 `token_type`，导致 EndUser token 可访问管理接口
