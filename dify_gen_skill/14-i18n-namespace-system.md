# Skill 14: `i18n-namespace-system` — 类型安全的 i18n 命名空间体系

**适用场景**: 需要支持多语言的大型前端项目。

**Dify 源码引用**:

- `web/i18n/en-US/` — 按功能域划分的翻译文件（app.ts / billing.ts / dataset.ts 等）
- `web/i18n/lib.server.ts` / `web/i18n/lib.client.ts` — 双模式 i18n 初始化
- `web/i18n/i18next-config.ts` — i18next 配置（24 种语言）

**Namespace 组织结构**:

```
web/i18n/
├── en-US/
│   ├── app.ts          # 应用管理相关
│   ├── billing.ts      # 计费相关
│   ├── dataset.ts      # 知识库相关
│   ├── workflow.ts     # 工作流相关
│   └── common.ts       # 通用文本
├── zh-Hans/            # 简体中文（同结构）
├── ja-JP/              # 日语（同结构）
├── lib.server.ts       # RSC 服务端 i18n
├── lib.client.ts       # 客户端 i18n
└── i18next-config.ts   # 语言列表 + 回退策略
```

**使用示例**:

```typescript
// 客户端组件
import { useTranslation } from 'react-i18next'

const MyComponent = () => {
  const { t } = useTranslation()
  return <h1>{t('app.title')}</h1>  // namespace.key 格式
}
```

**关键规则**:

- 所有用户可见字符串必须通过 `t()` 函数引用，禁止硬编码
- 翻译文件按功能域拆分 Namespace，单文件不超过 200 个 key
- 新增翻译 key 时先写 `en-US`，其他语言由翻译流程补充
- 使用 `#i18n` 条件导入区分 RSC 和客户端模式

**反模式**:

- ❌ 在组件中硬编码 `"保存"` / `"Save"` 等文本
- ❌ 所有翻译堆在一个 `common.ts` 中（应按功能域拆分）
- ❌ 翻译 key 使用中文（应使用 `app.createSuccess` 格式）
