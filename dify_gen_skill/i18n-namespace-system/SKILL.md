---
name: i18n-namespace-system
description: Use this skill when implementing internationalization (i18n) with namespace-based organization, setting up multi-language support with i18next, or configuring RSC server-side and client-side dual-mode i18n. Triggers on keywords like "i18n", "internationalization", "multi-language", "i18next", "translation namespace".
version: 1.0.0
---

# 类型安全的 i18n 命名空间体系

## Overview

按功能域命名空间组织翻译文件，支持 24 种语言，RSC 服务端 + 客户端双模式。

**Keywords**: i18n, internationalization, i18next, namespace, translation

## Dify 源码引用

- `web/i18n/en-US/` — 按功能域划分的翻译文件
- `web/i18n/lib.server.ts` / `web/i18n/lib.client.ts` — 双模式初始化
- `web/i18n/i18next-config.ts` — i18next 配置

## Namespace 组织结构

```
web/i18n/
├── en-US/
│   ├── app.ts
│   ├── billing.ts
│   ├── dataset.ts
│   ├── workflow.ts
│   └── common.ts
├── zh-Hans/
├── ja-JP/
├── lib.server.ts
├── lib.client.ts
└── i18next-config.ts
```

## 使用示例

```typescript
import { useTranslation } from 'react-i18next'

const MyComponent = () => {
  const { t } = useTranslation()
  return <h1>{t('app.title')}</h1>
}
```

## 关键规则

- 所有用户可见字符串必须通过 `t()` 函数引用
- 翻译文件按功能域拆分，单文件不超过 200 个 key
- 新增 key 时先写 `en-US`，其他语言由翻译流程补充

## 反模式

- 在组件中硬编码文本
- 所有翻译堆在一个 `common.ts` 中
- 翻译 key 使用中文
