# Skill 1: `flask-ddd-scaffold` — Flask DDD 分层架构脚手架

**适用场景**: 新建 Flask 后端项目时，快速搭建 DDD 分层结构。

**Dify 源码引用**:

- `api/app_factory.py` — Application Factory 入口
- `api/extensions/` — Flask 扩展统一初始化
- `api/controllers/` → `api/services/` → `api/models/` — 分层调用链
- `api/core/` — 领域核心逻辑（与 Flask 框架解耦）

**标准目录结构**:

```
api/
├── app_factory.py          # create_app() 工厂函数
├── extensions/             # Flask 扩展初始化（每个扩展一个文件）
│   ├── ext_database.py     # SQLAlchemy 初始化
│   ├── ext_redis.py        # Redis 连接池
│   ├── ext_celery.py       # Celery 初始化
│   ├── ext_login.py        # Flask-Login 配置
│   └── ext_blueprints.py   # Blueprint 统一注册
├── controllers/            # HTTP 层：接收请求、参数校验、调用 Service
│   ├── console/            # 管理后台 API
│   ├── service_api/        # 对外服务 API
│   └── web/                # 终端用户 API
├── services/               # 业务逻辑层：编排领域操作
│   ├── app_service.py
│   └── errors/             # 服务层异常定义
├── core/                   # 领域核心层：不依赖 Flask
│   ├── model_runtime/      # LLM Provider 适配
│   ├── rag/                # RAG 管道
│   ├── workflow/           # 工作流引擎
│   └── errors/             # 领域异常定义
├── models/                 # 数据模型层：SQLAlchemy ORM
│   ├── account.py
│   ├── model.py
│   └── workflow.py
├── configs/                # 配置管理（Pydantic BaseSettings）
└── tasks/                  # Celery 异步任务
```

**关键规则**:

- Controller 只做参数校验和响应格式化，禁止包含业务逻辑
- Service 层编排多个领域操作，通过构造函数注入依赖
- Core 层不导入 Flask/SQLAlchemy，保持领域纯净
- Extensions 每个扩展独立文件，通过 `init_app(app)` 模式初始化

**反模式**:

- ❌ 在 Controller 中直接操作数据库 `db.session.query(...)`
- ❌ 在 Core 层导入 `from flask import request`
- ❌ 将所有扩展初始化写在一个文件中
