# Skill 15: `tailwind-component-system` — Tailwind + CVA 组件变体系统

**适用场景**: 需要构建可复用 UI 组件库的前端项目。

**Dify 源码引用**:

- `web/tailwind-common-config.ts` — 设计 Token 定义（颜色、字体、间距）
- `web/utils/classnames.ts` — `cn()` 工具函数（`clsx` + `tailwind-merge`）
- `web/app/components/base/button/` — CVA 变体组件示例

**cn() 工具函数**:

```typescript
// utils/classnames.ts
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export const cn = (...inputs: ClassValue[]) => twMerge(clsx(inputs))
```

**CVA 组件变体示例**:

```typescript
// components/base/button/index.tsx
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

interface ButtonProps extends VariantProps<typeof buttonVariants> {
  className?: string
}

const Button: FC<ButtonProps> = ({ variant, size, className, ...props }) => (
  <button className={cn(buttonVariants({ variant, size }), className)} {...props} />
)
```

**关键规则**:

- 优先使用 Tailwind 工具类，仅在 Tailwind 无法实现时使用 CSS Module
- 条件 class 必须通过 `cn()` 处理，禁止手动字符串拼接
- 组件的 `className` prop 放在 `cn()` 最后，确保外部可覆盖内部样式
- 设计 Token 统一在 `tailwind-common-config.ts` 定义，禁止硬编码颜色值

**反模式**:

- ❌ 使用 `${isActive ? 'text-blue' : 'text-gray'}` 模板字符串拼接 class
- ❌ 新建 `.module.css` 文件实现简单样式（应用 Tailwind）
- ❌ 组件内硬编码 `#1677FF` 等颜色值（应用设计 Token）
