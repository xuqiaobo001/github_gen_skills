# Dify Architecture Skills for Claude Code

从 [Dify](https://github.com/langgenius/dify) 开源 LLM 应用平台源码中提炼的 **17 个架构技能**，已转换为标准 Claude Code Skill 格式，可直接加载使用。

## 使用方式

将所需的 skill 目录复制到 `~/.claude/skills/` 下即可：

```bash
cp -r <skill-name>/ ~/.claude/skills/
```

## 技能列表

### Backend（后端 · 10 个）

| Skill | 说明 |
|-------|------|
| `flask-ddd-scaffold` | Flask DDD 分层架构脚手架，Application Factory 模式 |
| `multi-tenant-rbac` | 多租户 RBAC 权限体系，五级角色层级 |
| `jwt-multi-auth` | JWT / API Key / OAuth 多策略认证 |
| `repository-factory` | Repository + Factory 数据访问层抽象 |
| `celery-async-tasks` | Celery 异步任务框架，队列分离与重试策略 |
| `pydantic-config-management` | Pydantic BaseSettings 配置管理 |
| `flask-blueprint-api` | Flask Blueprint 多 API 域路由组织 |
| `otel-observability` | OpenTelemetry 全链路可观测性 |
| `domain-error-hierarchy` | 领域异常层次体系与全局错误处理 |
| `plugin-extension-system` | 插件化扩展体系，Registry + Strategy 模式 |

### Frontend（前端 · 5 个）

| Skill | 说明 |
|-------|------|
| `nextjs-enterprise-scaffold` | Next.js App Router 企业级脚手架 |
| `orpc-contract-first-api` | oRPC 契约优先类型安全 API |
| `multi-layer-state-management` | Context/Jotai/Zustand/TanStack Query 多层状态管理 |
| `i18n-namespace-system` | i18next 命名空间国际化体系 |
| `tailwind-component-system` | Tailwind + CVA 组件变体系统 |

### Infrastructure（基础设施 · 2 个）

| Skill | 说明 |
|-------|------|
| `docker-compose-fullstack` | Docker Compose 全栈部署编排 |
| `event-driven-architecture` | Blinker 信号事件驱动架构 |

## 目录结构

每个 skill 遵循标准 Claude Code Skill 格式：

```
<skill-name>/
├── SKILL.md        # 技能定义（含 YAML frontmatter）
└── LICENSE.txt     # Apache 2.0 许可证
```

## License

Apache 2.0
