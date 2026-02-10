---
name: nextjs-enterprise-scaffold
description: Use this skill when creating a new Next.js enterprise project with App Router, organizing Layout Groups, structuring API integration layers (contract/service/component), or setting up business-domain component organization. Triggers on keywords like "Next.js scaffold", "App Router", "Layout Groups", "enterprise frontend", "Next.js project structure".
version: 1.0.0
---

# Next.js 企业级项目脚手架

## Overview

基于 Next.js App Router 构建企业级前端项目，包含 Layout Groups 分组、三层 API 集成和按业务域组织组件。

**Keywords**: Next.js, App Router, Layout Groups, enterprise scaffold, project structure

## Dify 源码引用

- `web/app/` — App Router 路由组织
- `web/app/(commonLayout)/` / `web/app/(shareLayout)/` — Layout Groups 分组
- `web/app/components/base/` — 100+ 基础组件库
- `web/context/` — 全局状态 Provider 组织

## 标准目录结构

```
web/
├── app/
│   ├── (commonLayout)/         # 带侧边栏的主布局
│   │   ├── apps/
│   │   ├── datasets/
│   │   └── layout.tsx
│   ├── (shareLayout)/          # 分享页布局
│   ├── components/
│   │   ├── base/               # 基础 UI 组件
│   │   ├── app/                # 应用业务组件
│   │   └── workflow/           # 工作流组件
│   └── layout.tsx
├── contract/                   # oRPC API 契约
├── service/                    # TanStack Query hooks
├── context/                    # React Context Providers
├── hooks/                      # 全局通用 hooks
├── types/                      # TypeScript 类型
├── i18n/                       # 国际化资源
└── utils/                      # 工具函数
```

## 关键规则

- 使用 Layout Groups `()` 实现同级路由不同布局
- API 集成三层分离：`contract/` → `service/` → 组件
- 用户可见字符串必须走 i18n，禁止硬编码
- 使用 `output: "standalone"` 模式构建

## 反模式

- 在 `app/` 目录下直接放业务组件
- 页面组件直接调用 API
- 在组件中硬编码中文/英文字符串
