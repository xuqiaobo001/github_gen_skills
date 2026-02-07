# Skill 5: `celery-async-tasks` — Celery 异步任务处理框架

**适用场景**: 需要后台任务队列、定时任务、长耗时操作异步化的项目。

**Dify 源码引用**:

- `api/extensions/ext_celery.py` — Celery 初始化（Gevent 兼容、Sentinel、SSL/TLS）
- `api/tasks/` — 按业务域组织的异步任务
- `api/schedule/` — APScheduler 定时任务集成

**任务组织结构**:

```
api/tasks/
├── mail_task.py              # 邮件发送
├── clean_unused_datasets_task.py  # 数据清理
├── workflow_task.py          # 工作流异步执行
└── annotation/               # 标注相关任务
    └── enable_annotation_reply_task.py
```

**核心代码骨架**:

```python
# tasks/mail_task.py
@shared_task(queue="mail", bind=True, max_retries=3)
def send_invite_email(self, tenant_id: str, account_id: str):
    """发送邀请邮件 — 失败自动重试"""
    try:
        account = Account.query.get(account_id)
        MailService.send_invite(account.email, tenant_id)
    except SMTPException as e:
        raise self.retry(exc=e, countdown=60 * (self.request.retries + 1))

# Service 层调用方式
class AccountService:
    def invite_member(self, tenant_id: str, email: str):
        account = self._create_account(email)
        send_invite_email.delay(tenant_id, account.id)  # 异步投递
```

**关键规则**:

- 任务函数只接收可序列化参数（ID、字符串），不传 ORM 对象
- 使用 `bind=True` + `max_retries` 实现自动重试
- 按业务域划分 queue（`mail`、`dataset`、`workflow`），便于独立扩缩容
- Redis 同时作为 Broker 和 Backend

**反模式**:

- ❌ 任务参数传递 SQLAlchemy 模型对象（序列化失败）
- ❌ 所有任务共用默认 queue，无法按优先级调度
- ❌ 在任务中直接 `import app`，应通过 `shared_task` 延迟绑定
