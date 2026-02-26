---
title: Định nghĩa và Sử dụng Design Tokens qua tailwind.config.js
impact: CRITICAL
impactDescription: Sử dụng raw values thay vì design tokens dẫn đến UI không nhất quán và khiến việc thay đổi theme yêu cầu tìm và thay thế toàn bộ
tags: tailwind, design-tokens, colors, spacing, typography
---

## Định nghĩa và Sử dụng Design Tokens qua tailwind.config.js

Tất cả các giá trị trực quan (colors, spacing, font sizes, border radii, shadows) phải được định nghĩa trong `tailwind.config.js` và tham chiếu bằng tên trong class names — không bao giờ sử dụng arbitrary hardcoded values cho những thứ thuộc về design system.

**Sai:**

```typescript
// ❌ Hardcoded hex values và arbitrary px values
function PrimaryButton({ children }: { children: React.ReactNode }) {
  return (
    <button className="bg-[#1A73E8] text-[#FFFFFF] px-[16px] py-[8px] rounded-[6px]">
      {children}
    </button>
  )
}
```

**Đúng:**

```typescript
// ✅ Sử dụng tokens được định nghĩa trong tailwind.config.js
// tailwind.config.js:
// theme.extend.colors.primary.DEFAULT = '#1A73E8'
// theme.extend.colors.white = '#FFFFFF'
// theme.extend.borderRadius.md = '6px'

function PrimaryButton({ children }: { children: React.ReactNode }) {
  return (
    <button className={cx("bg-primary text-white", "px-[1rem] py-[0.5rem] rounded-md")}>
      {children}
    </button>
  )
}
```

Spacing không nằm trong default scale của Tailwind phải được biểu diễn bằng `rem` sử dụng bracket notation với tỷ lệ `1rem = 16px`. Ví dụ, `16px → px-[1rem]`, `24px → px-[1.5rem]`.

Luôn kiểm tra `tailwind.config.js` trước khi thêm color hoặc size token mới để tránh trùng lặp.
