# Skill 17: `event-driven-architecture` — 事件驱动架构

**适用场景**: 需要解耦业务副作用（发邮件、审计日志、通知等）的系统。

**Dify 源码引用**:

- `api/events/` — 领域事件定义与处理器注册
- `api/events/tenant_event.py` — 租户创建事件示例
- `api/events/app_event.py` — 应用生命周期事件

**核心代码骨架**:

```python
# 1. 事件定义
from blinker import signal

# 定义领域事件信号
tenant_was_created = signal("tenant-was-created")
app_was_created = signal("app-was-created")
app_was_deleted = signal("app-was-deleted")

# 2. 事件发布（在 Service 层）
class TenantService:
    def create_tenant(self, name: str) -> Tenant:
        tenant = Tenant(name=name)
        db.session.add(tenant)
        db.session.commit()
        # 发布事件，不关心谁处理
        tenant_was_created.send(tenant)
        return tenant

# 3. 事件订阅（独立注册，不侵入业务代码）
@tenant_was_created.connect
def on_tenant_created(sender, **kwargs):
    """发送欢迎邮件"""
    send_welcome_email.delay(sender.id)

@tenant_was_created.connect
def on_tenant_created_audit(sender, **kwargs):
    """记录审计日志"""
    AuditLog.create(action="tenant_created",
                    resource_id=sender.id)
```

**关键规则**:

- 事件发布者只负责发信号，不关心有多少订阅者
- 每个订阅者处理一个独立副作用，保持单一职责
- 耗时操作（邮件、外部 API）通过 Celery 异步处理
- 事件处理器在应用启动时自动注册（通过模块导入）

**反模式**:

- ❌ 在 Service 层直接调用邮件/日志/通知等副作用代码
- ❌ 事件处理器之间互相依赖执行顺序
- ❌ 在事件处理器中修改核心业务数据（应只处理副作用）
