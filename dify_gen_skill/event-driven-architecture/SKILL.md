---
name: event-driven-architecture
description: Use this skill when implementing event-driven patterns to decouple business side effects, using Blinker signals for domain events, or organizing event publishers and subscribers. Triggers on keywords like "event-driven", "domain event", "Blinker signal", "event subscriber", "side effect decoupling".
version: 1.0.0
---

# 事件驱动架构

## Overview

基于 Blinker 信号实现事件驱动架构，Service 层发布事件，独立订阅者处理副作用。

**Keywords**: event-driven, domain event, Blinker, signal, subscriber, side effect

## Dify 源码引用

- `api/events/` — 领域事件定义与处理器注册
- `api/events/tenant_event.py` — 租户创建事件示例
- `api/events/app_event.py` — 应用生命周期事件

## 核心代码骨架

```python
# 1. 事件定义
from blinker import signal

tenant_was_created = signal("tenant-was-created")
app_was_created = signal("app-was-created")

# 2. 事件发布（Service 层）
class TenantService:
    def create_tenant(self, name: str) -> Tenant:
        tenant = Tenant(name=name)
        db.session.add(tenant)
        db.session.commit()
        tenant_was_created.send(tenant)
        return tenant

# 3. 事件订阅（独立注册）
@tenant_was_created.connect
def on_tenant_created(sender, **kwargs):
    send_welcome_email.delay(sender.id)

@tenant_was_created.connect
def on_tenant_created_audit(sender, **kwargs):
    AuditLog.create(action="tenant_created",
                    resource_id=sender.id)
```

## 关键规则

- 事件发布者只负责发信号，不关心订阅者
- 每个订阅者处理一个独立副作用，保持单一职责
- 耗时操作通过 Celery 异步处理
- 事件处理器在应用启动时自动注册

## 反模式

- 在 Service 层直接调用邮件/日志/通知等副作用代码
- 事件处理器之间互相依赖执行顺序
- 在事件处理器中修改核心业务数据
