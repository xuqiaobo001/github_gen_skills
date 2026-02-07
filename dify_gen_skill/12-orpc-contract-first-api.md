# Skill 12: `orpc-contract-first-api` — oRPC 契约优先 API 集成

**适用场景**: 需要前后端类型安全 API 调用的 TypeScript 项目。

**Dify 源码引用**:

- `web/contract/base.ts` — 基础契约（`inputStructure: 'detailed'`）
- `web/contract/router.ts` — 路由组合与类型导出
- `web/contract/console/` — 按业务域组织的契约文件
- `web/service/fetch.ts` — 基于 Ky 的 HTTP 客户端（CSRF、401 刷新、SSE）

**契约定义示例**:

```typescript
// web/contract/console/billing.ts
import { base } from '../base'
import { type } from '@orpc/contract'

export const listInvoices = base
  .route({ path: '/billing/invoices', method: 'GET' })
  .input(type<{ query: { page?: number; status?: string } }>())
  .output(type<InvoiceListResponse>())

export const getInvoice = base
  .route({ path: '/billing/invoices/{invoiceId}', method: 'GET' })
  .input(type<{ params: { invoiceId: string } }>())
  .output(type<InvoiceDetail>())
```

**Service Hook 示例**:

```typescript
// web/service/use-billing.ts
const NAME_SPACE = 'billing'

export const useInvoices = (params: { page?: number }) => {
  return useQuery({
    ...consoleQuery.billing.listInvoices.queryOptions({
      input: { query: params },
    }),
  })
}

export const useInvalidBilling = () => useInvalid([NAME_SPACE])
```

**关键规则**:

- Input 结构始终使用 `{ params, query?, body? }` 格式
- 路由按 API 前缀嵌套：`/billing/*` → `billing: {}`
- 禁止使用 barrel files（`index.ts` 统一导出），直接从具体文件导入
- 每个 service 文件定义 `NAME_SPACE` 用于 Query Key 组织

**反模式**:

- ❌ 在组件中直接 `fetch()`，绕过契约类型检查
- ❌ 手动拼接 Query Key 字符串而不用 `consoleQuery.*.queryKey()`
- ❌ 将多个不相关的 API 放在同一个契约文件中
