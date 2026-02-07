# Skill 13: `multi-layer-state-management` — 多层状态管理体系

**适用场景**: 大型前端应用需要管理全局状态、服务端状态、表单状态等多种状态。

**Dify 源码引用**:

- `web/context/` — React Context Providers（全局配置、用户信息）
- `web/app/components/workflow/store/` — Zustand Store（工作流编辑器）
- `web/service/use-*.ts` — TanStack Query hooks（服务端状态）
- `web/app/components/base/form/` — TanStack React Form 集成

**状态分层选型指南**:

| 层级 | 技术 | 适用场景 | Dify 示例 |
|------|------|----------|-----------|
| L1 | React Context + `use-context-selector` | 全局配置、用户信息等低频变更 | `web/context/provider-context.ts` |
| L2 | Jotai | 轻量局部状态、组件间共享原子值 | 局部 UI 开关状态 |
| L3 | Zustand | 复杂交互状态、高频更新 | `workflow/store/` 编辑器状态 |
| L4 | TanStack Query | 服务端数据缓存、自动刷新 | `service/use-workflow.ts` |
| L5 | TanStack React Form | 表单状态、字段级验证 | `components/base/form/` |

**Zustand Store 示例**:

```typescript
// workflow/store/index.ts
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

**关键规则**:

- 优先使用低层级方案：能用 Context 解决的不用 Zustand
- TanStack Query 的 Query Key 必须命名空间化（`[NAME_SPACE, 'detail', id]`）
- Zustand Store 用于高频交互场景（拖拽、画布编辑），Context 用于低频全局配置
- 禁止在 Zustand Store 中存储服务端数据（应由 TanStack Query 管理）

**反模式**:

- ❌ 用 Zustand 缓存 API 返回数据（应用 TanStack Query）
- ❌ 用 React Context 管理高频更新状态（导致大范围重渲染）
- ❌ Query Key 不分命名空间，导致缓存失效范围不可控
