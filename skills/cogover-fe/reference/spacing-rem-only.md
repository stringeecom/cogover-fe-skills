---
title: Luôn Sử dụng rem, Không Bao Giờ Dùng px
impact: CRITICAL
impactDescription: Sử dụng px units bỏ qua font-size preferences của user và phá vỡ accessibility zoom; rem scale đúng với root font size
tags: spacing, rem, px, accessibility, tailwind
---

## Luôn Sử dụng rem, Không Bao Giờ Dùng px

Tất cả các giá trị dimension — padding, margin, width, height, gap, border-radius, font-size — phải sử dụng `rem`. Không bao giờ sử dụng `px` trực tiếp trong Tailwind bracket notation hoặc inline styles.

Tỷ lệ chuyển đổi: **1rem = 16px** → `rem = px ÷ 16`

> Ví dụ: `24px ÷ 16 = 1.5` → dùng `[1.5rem]`

Các chuyển đổi phổ biến:

| px | rem |
|----|-----|
| 2px | 0.125rem |
| 4px | 0.25rem |
| 6px | 0.375rem |
| 8px | 0.5rem |
| 10px | 0.625rem |
| 12px | 0.75rem |
| 14px | 0.875rem |
| 16px | 1rem |
| 20px | 1.25rem |
| 24px | 1.5rem |
| 28px | 1.75rem |
| 32px | 2rem |
| 40px | 2.5rem |
| 48px | 3rem |
| 56px | 3.5rem |
| 64px | 4rem |

**Sai:**

```typescript
// ❌ px trong bracket notation
<div className="p-[16px] mt-[8px] rounded-[6px] w-[320px]">
  <span className="text-[14px]">Label</span>
</div>
```

**Đúng:**

```typescript
// ✅ rem trong bracket notation
<div className={cx("p-[1rem] mt-[0.5rem]", "rounded-[0.375rem] w-[20rem]")}>
  <span className={cx("text-[0.875rem]")}>Label</span>
</div>
```

**Ngoại lệ — 1px borders và dividers:**

Border widths chính xác `1px` có thể giữ nguyên là `border` (Tailwind default) hoặc `border-[1px]` vì sub-rem values (0.0625rem) không thực tế. Đây là ngoại lệ duy nhất được chấp nhận.

```typescript
// ✅ 1px border là ngoại lệ duy nhất được chấp nhận
<div className={cx("border border-border")} />
```
