# Skill 11: `nextjs-enterprise-scaffold` — Next.js 企业级项目脚手架

**适用场景**: 新建大型 Next.js 前端项目。

**Dify 源码引用**:

- `web/app/` — App Router 路由组织
- `web/app/(commonLayout)/` / `web/app/(shareLayout)/` — Layout Groups 分组
- `web/app/components/base/` — 100+ 基础组件库
- `web/context/` — 全局状态 Provider 组织
- `web/next.config.mjs` — Standalone 输出模式配置

**标准目录结构**:

```
web/
├── app/                        # App Router 路由
│   ├── (commonLayout)/         # 带侧边栏的主布局
│   │   ├── apps/               # 应用管理页
│   │   ├── datasets/           # 知识库页
│   │   └── layout.tsx          # 共享布局组件
│   ├── (shareLayout)/          # 分享页布局（无侧边栏）
│   ├── components/             # 页面级组件
│   │   ├── base/               # 基础 UI 组件（Button, Modal, Input...）
│   │   ├── app/                # 应用相关业务组件
│   │   └── workflow/           # 工作流编辑器组件
│   └── layout.tsx              # 根布局
├── contract/                   # oRPC API 契约定义
├── service/                    # TanStack Query hooks + HTTP 客户端
├── context/                    # React Context Providers
├── hooks/                      # 全局通用 hooks
├── types/                      # TypeScript 类型定义
├── i18n/                       # 国际化资源文件
├── utils/                      # 工具函数
└── public/                     # 静态资源
```

**关键规则**:

- 使用 Layout Groups `()` 实现同级路由不同布局，不嵌套路由
- `components/base/` 只放通用 UI 组件，业务组件按功能域放在对应目录
- API 集成三层分离：`contract/`（类型契约）→ `service/`（数据获取）→ 组件（消费数据）
- 用户可见字符串必须走 `i18n/en-US/`，禁止硬编码文本
- 使用 `output: "standalone"` 模式构建，便于 Docker 部署

**反模式**:

- ❌ 在 `app/` 目录下直接放业务组件（应放在 `app/components/` 下）
- ❌ 页面组件直接调用 API（应通过 `service/` 层的 hooks）
- ❌ 在组件中硬编码中文/英文字符串
