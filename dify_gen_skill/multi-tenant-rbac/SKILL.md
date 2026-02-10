---
name: multi-tenant-rbac
description: Use this skill when implementing multi-tenant workspace isolation, role-based access control (RBAC), permission decorators, or tenant data isolation in a SaaS system. Triggers on keywords like "multi-tenant", "RBAC", "role permission", "workspace isolation", "tenant data filtering".
version: 1.0.0
---

# 多租户 RBAC 权限体系

## Overview

实现工作空间级租户隔离与五级角色层级权限控制，适用于任何需要工作空间隔离 + 角色权限控制的 SaaS 系统。

**Keywords**: multi-tenant, RBAC, role-based access control, workspace isolation, permission decorator

## Dify 源码引用

- `api/models/account.py` — `Tenant` / `TenantAccountJoin` / `TenantAccountRole` 模型
- `api/libs/login.py` — `@login_required` 等权限装饰器
- `api/extensions/ext_login.py` — 多用户类型统一认证加载

## 核心模型骨架

```python
class TenantAccountRole(str, enum.Enum):
    OWNER = "owner"
    ADMIN = "admin"
    EDITOR = "editor"
    NORMAL = "normal"
    DATASET_OPERATOR = "dataset_operator"

class Tenant(db.Model):
    id = db.Column(StringUUID, primary_key=True)
    name = db.Column(db.String(255), nullable=False)
    plan = db.Column(db.String(255), default="sandbox")
    status = db.Column(db.String(255), default="active")

class TenantAccountJoin(db.Model):
    """多对多关联表：用户 ↔ 租户 + 角色"""
    tenant_id = db.Column(StringUUID, db.ForeignKey("tenants.id"))
    account_id = db.Column(StringUUID, db.ForeignKey("accounts.id"))
    role = db.Column(db.String(16), nullable=False)
```

## 权限装饰器链

```python
@app.route("/api/admin/settings")
@login_required
@account_initialization_required
@is_admin_or_owner_required
def admin_settings():
    ...
```

## 关键规则

- 所有业务模型必须携带 `tenant_id` 字段实现数据隔离
- 查询时必须附加 `filter_by(tenant_id=current_user.current_tenant_id)`
- 角色检查通过装饰器实现，禁止在业务代码中硬编码角色判断

## 反模式

- 查询数据时忘记加 `tenant_id` 过滤，导致跨租户数据泄露
- 在 Service 层用 `if role == "admin"` 硬编码权限判断
