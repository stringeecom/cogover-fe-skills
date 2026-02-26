---
title: Sử dụng cx() cho className với Grouped Classes
impact: HIGH
impactDescription: Concatenating class strings bằng template literals hoặc toán tử + làm cho classes khó scan, merge conflicts đau đầu, và conditional classes dễ lỗi
tags: tailwind, cx, classnames, className, conditional
---

## Sử dụng cx() cho className với Grouped Classes

Luôn sử dụng utility function `cx` để compose Tailwind classes. Không bao giờ dùng string concatenation, template literals, hoặc ternary expressions bên trong className string.

Nhóm các classes liên quan vào một **argument duy nhất** của `cx`. Mỗi argument nên đại diện cho một concern (layout, typography, color, interaction, v.v.). Tránh dump tất cả classes vào một string dài.

### Import

Luôn import `cx` ở đầu file trước khi sử dụng:

```typescript
import { cx } from '@/utils/cx'  // điều chỉnh path tới cx location của project
```

### Sai:

```typescript
// ❌ String concatenation
<div className={"flex items-center " + (isActive ? "bg-primary-main" : "bg-surface-default")}>

// ❌ Template literal
<div className={`flex items-center p-[1rem] ${isActive ? "text-primary-main" : "text-text-muted"}`}>

// ❌ Tất cả classes nhồi nhét vào một argument — khó đọc
<div className={cx("flex items-center justify-between p-[1rem] m-[0.5rem] rounded-[0.5rem] border border-border-default bg-surface-raised prose-body2 text-text-primary hover:bg-surface-hover cursor-pointer")}>

// ❌ cx không được import — sẽ throw lỗi runtime
<div className={cx("flex p-[1rem]")}>
```

### Đúng:

```typescript
import { cx } from '@/utils/cx'

// ✅ Mỗi argument là một concern
<div className={cx(
  "flex items-center justify-between",          // layout
  "p-[1rem] m-[0.5rem] rounded-[0.5rem]",      // spacing & shape
  "border border-border-default bg-surface-raised", // border & background
  "prose-body2 text-text-primary",              // typography & color
  "hover:bg-surface-hover cursor-pointer",      // interaction
)}>

// ✅ Conditional classes dùng object syntax
<div className={cx(
  "flex items-center gap-[0.5rem]",
  "prose-body2",
  {
    "text-primary-main bg-primary-subtle": isActive,
    "text-text-muted bg-surface-default": !isActive,
  },
)}>

// ✅ Multiple conditions
<button className={cx(
  "inline-flex items-center justify-center rounded-[0.375rem]",
  "px-[1rem] py-[0.5rem]",
  "prose-label1 transition-colors",
  {
    "bg-primary-main text-white hover:bg-primary-dark": variant === 'primary',
    "bg-transparent border border-border-default text-text-primary": variant === 'secondary',
    "opacity-50 cursor-not-allowed pointer-events-none": disabled,
  },
)}>
```

### Hướng dẫn nhóm:

| Argument | Classes thuộc về nhau |
|----------|------------------------------|
| 1st | Layout: `flex`, `grid`, `block`, `items-*`, `justify-*`, `gap-*` |
| 2nd | Spacing & shape: `p-*`, `m-*`, `w-*`, `h-*`, `rounded-*` |
| 3rd | Visual: `bg-*`, `border-*`, `shadow-*` |
| 4th | Typography: `prose-*`, `text-*` (color), `truncate` |
| 5th | Interaction: `hover:*`, `focus:*`, `cursor-*`, `transition-*` |
| Object | Conditional classes luôn dùng object syntax `{ "class": condition }` |

Số lượng arguments linh hoạt — mục tiêu là khả năng đọc (readability), không phải tuân thủ nghiêm ngặt bảng này.
