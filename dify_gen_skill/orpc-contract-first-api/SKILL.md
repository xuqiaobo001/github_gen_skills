---
name: orpc-contract-first-api
description: Use this skill when implementing contract-first API design with oRPC, defining type-safe API contracts in TypeScript, integrating TanStack Query with typed API hooks, or structuring frontend API layers. Triggers on keywords like "oRPC", "contract-first API", "type-safe API", "TanStack Query integration", "API contract".
version: 1.0.0
license: Apache 2.0, see LICENSE.txt
---

# oRPC 契约优先 API 集成

## Overview

基于 oRPC 实现契约优先的类型安全 API 定义，标准化输入结构 {params, query, body}，集成 TanStack Query。

**Keywords**: oRPC, contract-first, type-safe API, TanStack Query, API contract

## Dify 源码引用

- `web/contract/base.ts` — 基础契约
- `web/contract/router.ts` — 路由组合与类型导出
- `web/contract/console/` — 按业务域组织的契约文件
- `web/service/fetch.ts` — 基于 Ky 的 HTTP 客户端

## 契约定义示例

```typescript
import { base } from '../base'
import { type } from '@orpc/contract'

export const listInvoices = base
  .route({ path: '/billing/invoices', method: 'GET' })
  .input(type<{ query: { page?: number; status?: string } }>())
  .output(type<InvoiceListResponse>())
```

## Service Hook 示例

```typescript
const NAME_SPACE = 'billing'

export const useInvoices = (params: { page?: number }) => {
  return useQuery({
    ...consoleQuery.billing.listInvoices.queryOptions({
      input: { query: params },
    }),
  })
}
```

## 关键规则

- Input 结构始终使用 `{ params, query?, body? }` 格式
- 路由按 API 前缀嵌套
- 禁止使用 barrel files，直接从具体文件导入
- 每个 service 文件定义 `NAME_SPACE` 用于 Query Key 组织

## 反模式

- 在组件中直接 `fetch()`，绕过契约类型检查
- 手动拼接 Query Key 字符串
- 将多个不相关的 API 放在同一个契约文件中
