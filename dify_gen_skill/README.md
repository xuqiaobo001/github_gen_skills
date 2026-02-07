# Dify Architecture Skills

从 [Dify](https://github.com/langgenius/dify) 开源 LLM 应用平台源码中提炼的 **17 个架构技能**，涵盖后端、前端和基础设施三大领域。每个技能以独立 Markdown 文件呈现，包含核心模式、代码示例、关键规则与反模式，可作为构建企业级 LLM 应用的参考架构与最佳实践指南。

## 目录结构

```
dify_gen_skill/
├── backend_skills/          # 后端技能 (10 个)
├── frontend_skills/         # 前端技能 (5 个)
├── infra_skills/            # 基础设施技能 (2 个)
└── README.md
```

## 技能总览

### Backend Skills（后端 · 10 个）

| # | 文件 | 技能名称 | 说明 |
|---|------|---------|------|
| 01 | `01-flask-ddd-scaffold.md` | Flask DDD 分层脚手架 | 基于领域驱动设计的 Flask 项目分层架构：Controllers → Services → Core → Models，Application Factory 模式 |
| 02 | `02-multi-tenant-rbac.md` | 多租户 RBAC 权限体系 | Workspace 级租户隔离，五级角色层级（Owner/Admin/Editor/Normal/Dataset Operator），装饰器权限校验 |
| 03 | `03-jwt-multi-auth.md` | JWT 多策略认证 | 支持 JWT / API Key / OAuth 多种认证方式，Token 提取优先级：Header → Cookie → Query |
| 04 | `04-repository-factory.md` | Repository Factory 模式 | 抽象数据访问层，工厂模式动态选择实现，支持 15+ 向量数据库（Qdrant、Weaviate、Milvus 等） |
| 05 | `05-celery-async-tasks.md` | Celery 异步任务框架 | 按业务域组织任务，指数退避自动重试，队列分离（mail/dataset/workflow） |
| 06 | `06-pydantic-config-management.md` | Pydantic 配置管理 | 类型安全的环境变量驱动配置，Mixin 按功能域组织，冻结配置防止运行时修改 |
| 07 | `07-flask-blueprint-api.md` | Flask Blueprint 多 API 组织 | 5 种 Blueprint 类型（console/service_api/web/inner_api），独立 URL 前缀与 CORS 策略 |
| 08 | `08-otel-observability.md` | OpenTelemetry 全链路可观测 | 统一 Instrumentor 初始化（Flask/SQLAlchemy/Celery/Redis），请求级 Trace/Span ID 注入 |
| 09 | `09-domain-error-hierarchy.md` | 领域错误层级体系 | 层次化异常结构（DifyBaseError 基类），全局异常处理器映射 HTTP 状态码 |
| 10 | `10-plugin-extension-system.md` | 插件扩展系统 | 抽象基类定义插件契约，Registry 模式管理插件，Strategy 模式实现工具，Provider Adapter 适配 LLM 供应商 |

### Frontend Skills（前端 · 5 个）

| # | 文件 | 技能名称 | 说明 |
|---|------|---------|------|
| 11 | `11-nextjs-enterprise-scaffold.md` | Next.js 企业级脚手架 | App Router + Layout Groups，三层 API 集成（contract → service → component），按业务域组织组件 |
| 12 | `12-orpc-contract-first-api.md` | oRPC 契约优先 API | 契约优先的类型安全 API 定义，标准化输入结构 {params, query, body}，TanStack Query 集成 |
| 13 | `13-multi-layer-state-management.md` | 多层状态管理 | 五层状态层级：Context → Jotai → Zustand → TanStack Query → React Form |
| 14 | `14-i18n-namespace-system.md` | 类型安全 i18n 命名空间 | 按功能域命名空间组织，支持 24 种语言，RSC 服务端 + 客户端双模式 |
| 15 | `15-tailwind-component-system.md` | Tailwind + CVA 组件体系 | CVA 管理组件变体，cn() 工具函数（clsx + tailwind-merge），Design Token 统一管理 |

### Infrastructure Skills（基础设施 · 2 个）

| # | 文件 | 技能名称 | 说明 |
|---|------|---------|------|
| 16 | `16-docker-compose-fullstack.md` | Docker Compose 全栈部署 | 服务分离（API/Worker/Web/Nginx），中间件独立编排，Nginx 统一入口，SSRF 代理过滤 |
| 17 | `17-event-driven-architecture.md` | 事件驱动架构 | Blinker 信号事件系统，Service 层发布事件，独立订阅者处理副作用，Celery 异步处理长耗时操作 |

## 核心架构原则

这 17 个技能贯穿以下架构原则：

- **分层架构** — HTTP 层、业务逻辑层、领域层职责清晰分离
- **依赖注入** — 构造器注入实现松耦合
- **工厂模式** — 基于配置动态选择实现
- **单一职责** — 每个组件只有一个变更理由
- **类型安全** — Python 端 Pydantic，TypeScript 端严格类型
- **配置驱动** — 环境变量驱动，杜绝硬编码
- **错误层级** — 层次化异常结构 + 全局处理器
- **可观测性** — 分布式追踪与结构化日志贯穿全链路
- **可扩展性** — 异步处理、水平扩展、多租户支持

## 使用方式

每个技能文件均为独立的 Markdown 文档，可以：

1. **作为学习材料** — 逐个阅读，从基础脚手架（01）到高级模式（17）渐进学习
2. **作为 AI Coding 技能** — 将技能文件加载到 AI 编程助手（如 Claude Code、Cursor 等）的上下文中，辅助生成符合 Dify 架构风格的代码
3. **作为架构参考** — 在构建类似的 LLM 应用平台时，参考其中的设计模式与最佳实践

## 来源

所有技能均从 [Dify](https://github.com/langgenius/dify) 开源项目源码中提炼，Dify 是一个开源的 LLM 应用开发平台，支持从 Agent 构建到 AI Workflow 编排、RAG 检索、模型管理等能力，帮助开发者快速构建生产级 AI 应用。
