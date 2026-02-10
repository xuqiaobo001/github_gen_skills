---
name: tailwind-component-system
description: Use this skill when building a reusable UI component library with Tailwind CSS and CVA (class-variance-authority), implementing component variants, or setting up design tokens. Triggers on keywords like "Tailwind component", "CVA", "component variants", "design tokens", "cn() utility".
version: 1.0.0
---

# Tailwind + CVA 组件变体系统

## Overview

基于 CVA 管理组件变体，cn() 工具函数处理条件样式，Design Token 统一管理。

**Keywords**: Tailwind CSS, CVA, class-variance-authority, component variants, design tokens

## Dify 源码引用

- `web/tailwind-common-config.ts` — 设计 Token 定义
- `web/utils/classnames.ts` — `cn()` 工具函数
- `web/app/components/base/button/` — CVA 变体组件示例

## cn() 工具函数

```typescript
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export const cn = (...inputs: ClassValue[]) => twMerge(clsx(inputs))
```

## CVA 组件变体示例

```typescript
import { cva, type VariantProps } from 'class-variance-authority'

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-lg font-medium transition-colors',
  {
    variants: {
      variant: {
        primary: 'bg-primary-600 text-white hover:bg-primary-700',
        secondary: 'bg-white text-gray-700 border border-gray-200 hover:bg-gray-50',
        ghost: 'text-gray-500 hover:bg-gray-100',
      },
      size: {
        sm: 'h-7 px-3 text-xs',
        md: 'h-9 px-4 text-sm',
        lg: 'h-11 px-6 text-base',
      },
    },
    defaultVariants: { variant: 'primary', size: 'md' },
  }
)

const Button: FC<ButtonProps> = ({ variant, size, className, ...props }) => (
  <button className={cn(buttonVariants({ variant, size }), className)} {...props} />
)
```

## 关键规则

- 优先使用 Tailwind 工具类
- 条件 class 必须通过 `cn()` 处理，禁止手动拼接
- `className` prop 放在 `cn()` 最后，确保外部可覆盖
- 设计 Token 统一在 config 中定义，禁止硬编码颜色值

## 反模式

- 使用模板字符串拼接 class
- 新建 `.module.css` 实现简单样式
- 组件内硬编码颜色值
