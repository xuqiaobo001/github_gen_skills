# Auto Gen Skills

从开源项目源码与云服务官方文档中自动提炼的 **Claude Code Skills** 集合。每个技能以结构化 Markdown 文件呈现，可供开发者学习参考，也可作为 AI 编程助手的上下文知识加载使用。

## 项目理念

优秀的开源项目蕴含大量经过生产验证的架构模式与工程实践，云服务文档则包含丰富的运维操作指南。本项目通过 AI 辅助分析，将这些知识提炼为独立、可复用的 Claude Code Skill 文件，降低学习门槛，提升开发效率。

## 目录结构

```
auto_gen_skills/
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
│   └── event-driven-architecture/       #   事件驱动架构
├── ascend_skill_gen/                    # 华为 Ascend NPU & ModelArts（25 个技能）
│   ├── ma_standard_skills/              #   ModelArts Standard Skills（9 个）
│   ├── ma_lite_devserver_skills/        #   ModelArts Lite Server Skills（9 个）
│   └── ma_lite_cluster_skills/          #   ModelArts Lite Cluster Skills（7 个）
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
| [Huawei Ascend / ModelArts](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/) | `ascend_skill_gen/` | 25 | NPU 资源池管理、Notebook 开发、自定义镜像构建、GPU→NPU 代码迁移、分布式训练 (HCCL/Volcano)、高可靠训练、推理部署、监控告警、Workflow 流水线、Lite Server 运维、Lite Cluster K8s 管理、插件管理 |

> 后续将持续收录更多项目的技能。

## 技能文件规范

每个技能文件遵循 Claude Code Skill 标准格式，根据来源不同侧重点有所区别：

### 开源项目架构技能（如 Dify）

- **核心模式** — 关键设计模式与架构决策
- **代码示例** — 从源码中提取的真实代码片段
- **关键规则** — 使用该模式时必须遵守的约束
- **反模式** — 常见的错误用法与应避免的做法

### 云服务运维技能（如 Ascend / ModelArts）

- **Overview** — 技能概述与适用范围
- **When to Use** — 触发场景列表
- **操作指南** — 分步骤操作说明、命令示例、配置模板
- **Reference Documentation** — 官方文档链接

## 在 Claude Code 中安装部署

### 前置条件

- 已安装 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI 工具
- 项目目录下已初始化 `.claude/` 配置目录（首次运行 `claude` 命令时自动创建）

### Skills 发现机制

Claude Code 从以下位置自动发现并加载 Skills：

| 作用域 | 路径 | 说明 |
|--------|------|------|
| 项目级 | `<project-root>/.claude/skills/<skill-name>/SKILL.md` | 仅当前项目可用，可通过 Git 共享给团队 |
| 用户级 | `~/.claude/skills/<skill-name>/SKILL.md` | 所有项目可用，仅限当前用户 |

Claude Code 采用三级渐进加载策略：
1. **元数据加载** — 启动时仅读取所有 Skills 的 `name` 和 `description` 字段
2. **按需加载** — 当对话内容匹配某个 Skill 的描述时，加载完整 `SKILL.md`
3. **资源加载** — 如 Skill 目录下包含 `scripts/`、`references/`、`assets/` 子目录，按需加载

### 方式一：复制到项目目录（推荐）

适合团队协作，Skills 随项目代码一起版本管理。

```bash
# 进入你的项目根目录
cd /path/to/your/project

# 创建 skills 目录
mkdir -p .claude/skills

# 克隆本仓库
git clone https://github.com/xuqiaobo001/auto_gen_skills.git /tmp/auto_gen_skills

# 按需复制单个 skill
cp -r /tmp/auto_gen_skills/dify_gen_skill/flask-ddd-scaffold .claude/skills/

# 或批量复制某个项目的全部 skills
cp -r /tmp/auto_gen_skills/dify_gen_skill/* .claude/skills/
cp -r /tmp/auto_gen_skills/ascend_skill_gen/ma_standard_skills/* .claude/skills/
cp -r /tmp/auto_gen_skills/ascend_skill_gen/ma_lite_devserver_skills/* .claude/skills/
cp -r /tmp/auto_gen_skills/ascend_skill_gen/ma_lite_cluster_skills/* .claude/skills/
```

### 方式二：安装到用户目录（全局可用）

适合个人使用，所有项目共享同一套 Skills。

```bash
mkdir -p ~/.claude/skills

# 复制 Dify 架构技能
cp -r /tmp/auto_gen_skills/dify_gen_skill/* ~/.claude/skills/

# 复制 Ascend / ModelArts 运维技能
cp -r /tmp/auto_gen_skills/ascend_skill_gen/ma_standard_skills/* ~/.claude/skills/
cp -r /tmp/auto_gen_skills/ascend_skill_gen/ma_lite_devserver_skills/* ~/.claude/skills/
cp -r /tmp/auto_gen_skills/ascend_skill_gen/ma_lite_cluster_skills/* ~/.claude/skills/
```

### 方式三：符号链接（便于同步更新）

克隆仓库后通过符号链接引用，`git pull` 即可获取最新 Skills。

```bash
# 克隆仓库到固定位置
git clone https://github.com/xuqiaobo001/auto_gen_skills.git ~/auto_gen_skills

# 进入项目目录
cd /path/to/your/project
mkdir -p .claude/skills

# 批量链接所有 skills
for dir in ~/auto_gen_skills/dify_gen_skill/*/; do
  ln -sf "$dir" .claude/skills/$(basename "$dir")
done
for dir in ~/auto_gen_skills/ascend_skill_gen/ma_*/*/; do
  ln -sf "$dir" .claude/skills/$(basename "$dir")
done
```

### 验证安装

启动 Claude Code，输入 `/` 查看已加载的 Skills 列表：

```
$ claude
> /flask-ddd-scaffold
> /ascend-distributed-training
> /lite-cluster-plugin
```

Claude 也会根据对话内容自动匹配并加载相关 Skill，无需手动调用。

### 注意事项

- 每个 skill 目录必须包含 `SKILL.md` 文件（大写），这是 Claude Code 识别 skill 的唯一入口
- YAML frontmatter 中 `name` 和 `description` 为必需字段
- `name` 字段要求：小写字母、数字、连字符，最长 64 字符
- 建议按需安装，避免一次性加载过多 Skills 占用上下文窗口
- 更新 Skills 后需重启 Claude Code 会话才能生效
- 建议将 `.claude/skills/` 加入项目 `.gitignore`（如不希望 Skills 进入版本控制）

## 其他使用方式

除了在 Claude Code 中加载，本仓库的 Skills 还可以：

- **作为学习材料** — 阅读各项目的技能文件，系统性学习架构模式与云服务运维实践
- **作为架构参考** — 在构建新项目或管理云资源时，参考已收录的设计模式与操作指南

## 贡献

欢迎为本仓库贡献新的技能提炼，请遵循现有的目录组织方式：

1. 创建 `<project>_gen_skill/` 目录
2. 每个技能创建独立子目录，包含 `SKILL.md` 和 `LICENSE.txt`
3. `SKILL.md` 使用 YAML frontmatter（name, description, license）
4. 为项目目录编写 `README.md` 说明

## 许可证

所有 Skills 均采用 [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0) 许可证。
