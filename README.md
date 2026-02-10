# GitHub Gen Skills

从知名开源项目源码中自动提炼的 **架构技能（Architecture Skills）** 集合。每个技能以结构化 Markdown 文件呈现，包含核心设计模式、代码示例、关键规则与反模式，可供开发者学习参考，也可作为 AI 编程助手的上下文知识加载使用。

## 项目理念

优秀的开源项目蕴含大量经过生产验证的架构模式与工程实践，但这些知识往往分散在代码各处，难以系统性获取。本项目通过 AI 辅助分析源码，将这些隐性知识提炼为独立、可复用的技能文件，降低学习门槛，提升开发效率。

## 目录结构

```
github_gen_skill/
├── dify_gen_skill/                      # Dify — 开源 LLM 应用平台（17 个技能）
│   ├── flask-ddd-scaffold/              #   Flask DDD 分层架构脚手架
│   ├── multi-tenant-rbac/               #   多租户 RBAC 权限体系
│   ├── jwt-multi-auth/                  #   JWT 多策略认证
│   ├── repository-factory/              #   Repository Factory 数据访问层
│   ├── celery-async-tasks/              #   Celery 异步任务框架
│   ├── pydantic-config-management/      #   Pydantic 配置管理
│   ├── flask-blueprint-api/             #   Flask Blueprint 多 API 域
│   ├── otel-observability/              #   OpenTelemetry 可观测性
│   ├── domain-error-hierarchy/          #   领域异常层次体系
│   ├── plugin-extension-system/         #   插件化扩展体系
│   ├── nextjs-enterprise-scaffold/      #   Next.js 企业级脚手架
│   ├── orpc-contract-first-api/         #   oRPC 契约优先 API
│   ├── multi-layer-state-management/    #   多层状态管理
│   ├── i18n-namespace-system/           #   i18n 命名空间体系
│   ├── tailwind-component-system/       #   Tailwind + CVA 组件系统
│   ├── docker-compose-fullstack/        #   Docker Compose 全栈部署
│   ├── event-driven-architecture/       #   事件驱动架构
│   └── README.md
├── ascend_skill_gen/                    # 华为 ModelArts 相关技能
└── README.md
```

每个 skill 目录遵循标准 Claude Code Skill 格式：

```
<skill-name>/
├── SKILL.md        # 技能定义（含 YAML frontmatter: name, description, version, license）
└── LICENSE.txt     # Apache 2.0 许可证
```

## 已收录项目

| 项目 | 目录 | 技能数 | 涵盖领域 |
|------|------|--------|----------|
| [Dify](https://github.com/langgenius/dify) | `dify_gen_skill/` | 17 | Flask DDD 架构、多租户 RBAC、JWT 认证、Repository Factory、Celery 异步任务、Pydantic 配置、Blueprint API、OpenTelemetry 可观测、领域错误层级、插件系统、Next.js 脚手架、oRPC 契约 API、多层状态管理、i18n、Tailwind 组件、Docker Compose 部署、事件驱动架构 |

> 后续将持续收录更多开源项目的架构技能。

## 技能文件规范

每个技能文件遵循统一结构：

- **核心模式** — 该技能涉及的关键设计模式与架构决策
- **代码示例** — 从源码中提取的真实代码片段
- **关键规则** — 使用该模式时必须遵守的约束
- **反模式** — 常见的错误用法与应避免的做法
- **适用场景** — 何时应该使用该技能

## 使用方式

### 1. 作为学习材料

按编号顺序阅读各项目的技能文件，从基础架构到高级模式渐进学习。

### 2. 作为 AI Coding 技能

将技能文件加载到 AI 编程助手（Claude Code、Cursor、GitHub Copilot 等）的上下文中，辅助生成符合对应项目架构风格的代码。

### 3. 作为架构参考

在构建新项目时，参考已收录项目中经过生产验证的设计模式与最佳实践。

## 贡献

欢迎为本仓库贡献新的项目技能提炼，请遵循现有的目录组织方式：

1. 创建 `<project>_gen_skill/` 目录
2. 按领域分类创建子目录（如 `backend_skills/`、`frontend_skills/`）
3. 技能文件使用编号前缀命名（如 `01-xxx.md`）
4. 为子目录编写 `README.md` 说明
