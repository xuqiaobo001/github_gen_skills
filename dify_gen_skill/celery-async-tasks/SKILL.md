---
name: celery-async-tasks
description: Use this skill when implementing async task queues with Celery, organizing background tasks by business domain, setting up task retry strategies, or separating task queues for different priorities. Triggers on keywords like "Celery", "async task", "background job", "task queue", "retry strategy".
version: 1.0.0
license: Apache 2.0, see LICENSE.txt
---

# Celery 异步任务处理框架

## Overview

基于 Celery 构建按业务域组织的异步任务框架，支持指数退避重试和队列分离。

**Keywords**: Celery, async task, background job, task queue, retry, worker

## Dify 源码引用

- `api/extensions/ext_celery.py` — Celery 初始化
- `api/tasks/` — 按业务域组织的异步任务
- `api/schedule/` — APScheduler 定时任务集成

## 任务组织结构

```
api/tasks/
├── mail_task.py
├── clean_unused_datasets_task.py
├── workflow_task.py
└── annotation/
    └── enable_annotation_reply_task.py
```

## 核心代码骨架

```python
@shared_task(queue="mail", bind=True, max_retries=3)
def send_invite_email(self, tenant_id: str, account_id: str):
    try:
        account = Account.query.get(account_id)
        MailService.send_invite(account.email, tenant_id)
    except SMTPException as e:
        raise self.retry(exc=e, countdown=60 * (self.request.retries + 1))

class AccountService:
    def invite_member(self, tenant_id: str, email: str):
        account = self._create_account(email)
        send_invite_email.delay(tenant_id, account.id)
```

## 关键规则

- 任务函数只接收可序列化参数（ID、字符串），不传 ORM 对象
- 使用 `bind=True` + `max_retries` 实现自动重试
- 按业务域划分 queue（`mail`、`dataset`、`workflow`）
- Redis 同时作为 Broker 和 Backend

## 反模式

- 任务参数传递 SQLAlchemy 模型对象（序列化失败）
- 所有任务共用默认 queue，无法按优先级调度
- 在任务中直接 `import app`，应通过 `shared_task` 延迟绑定
