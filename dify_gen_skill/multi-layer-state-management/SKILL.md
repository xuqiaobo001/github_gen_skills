---
name: multi-layer-state-management
description: Use this skill when designing frontend state management architecture, choosing between Context/Jotai/Zustand/TanStack Query/React Form, or organizing state by update frequency and scope. Triggers on keywords like "state management", "Zustand", "TanStack Query", "React Context", "Jotai", "frontend state".
version: 1.0.0
---

# 多层状态管理体系

## Overview

五层状态管理选型指南：Context → Jotai → Zustand → TanStack Query → React Form。

**Keywords**: state management, Zustand, TanStack Query, React Context, Jotai, React Form

## Dify 源码引用

- `web/context/` — React Context Providers
- `web/app/components/workflow/store/` — Zustand Store
- `web/service/use-*.ts` — TanStack Query hooks
- `web/app/components/base/form/` — TanStack React Form

## 状态分层选型指南

| 层级 | 技术 | 适用场景 |
|------|------|----------|
| L1 | React Context | 全局配置、用户信息等低频变更 |
| L2 | Jotai | 轻量局部状态、组件间共享原子值 |
| L3 | Zustand | 复杂交互状态、高频更新 |
| L4 | TanStack Query | 服务端数据缓存、自动刷新 |
| L5 | TanStack React Form | 表单状态、字段级验证 |

## Zustand Store 示例

```typescript
import { create } from 'zustand'

interface WorkflowState {
  nodes: Node[]
  edges: Edge[]
  selectedNodeId: string | null
  updateNode: (id: string, data: Partial<Node>) => void
  selectNode: (id: string | null) => void
}

export const useWorkflowStore = create<WorkflowState>((set) => ({
  nodes: [],
  edges: [],
  selectedNodeId: null,
  updateNode: (id, data) => set((state) => ({
    nodes: state.nodes.map(n => n.id === id ? { ...n, ...data } : n),
  })),
  selectNode: (id) => set({ selectedNodeId: id }),
}))
```

## 关键规则

- 优先使用低层级方案：能用 Context 解决的不用 Zustand
- TanStack Query 的 Query Key 必须命名空间化
- Zustand 用于高频交互，Context 用于低频全局配置
- 禁止在 Zustand Store 中存储服务端数据

## 反模式

- 用 Zustand 缓存 API 返回数据
- 用 React Context 管理高频更新状态
- Query Key 不分命名空间
