---
title: Cách sử dụng SmartTooltip Component của ui-kit
impact: HIGH
impactDescription: Sử dụng Tooltip thay vì SmartTooltip cho truncated text khiến tooltip hiện cả khi text vừa vặn, gây nhiễu
tags: tooltip, smart-tooltip, truncate, line-clamp, overflow, ui-kit
---

## Cách sử dụng SmartTooltip Component của ui-kit

Đường dẫn component: `ui-kit/src/components/SmartTooltip`

`SmartTooltip` wrap `Tooltip` và tự động enable/disable nó dựa trên việc content của target element có bị overflow hay không. Sử dụng `useResizeObserver` nên nó phản ứng với container resizes.

Props:
- `children`: `({ getRef }) => ReactElement` — render prop (bắt buộc)
- `multipleLines?: boolean` — mặc định `false`
- Tất cả `TooltipProps` trừ `children`

### RULE-SMTTP-01: Sử dụng `SmartTooltip` thay vì `Tooltip` khi text có thể bị truncate

**Sai:**
```tsx
// ❌ Tooltip luôn hiện, ngay cả khi text không bị cắt
<Tooltip content={name}>
  <p className="truncate w-[12.5rem]">{name}</p>
</Tooltip>
```

**Đúng:**
```tsx
// ✅ Tooltip chỉ hiện khi text thực sự bị truncate
<SmartTooltip content={name}>
  {({ getRef }) => (
    <p ref={getRef} className="truncate w-[12.5rem]">{name}</p>
  )}
</SmartTooltip>
```

### RULE-SMTTP-02: Attach `getRef` vào overflowing element, không phải wrapper

**Sai:**
```tsx
// ❌ Ref trên wrapper — overflow check sai
<SmartTooltip content={text}>
  {({ getRef }) => (
    <div ref={getRef} className="overflow-hidden">
      <span className="truncate">{text}</span>
    </div>
  )}
</SmartTooltip>
```

**Đúng:**
```tsx
// ✅ Ref trên truncated element
<SmartTooltip content={text}>
  {({ getRef }) => (
    <div className="overflow-hidden">
      <span ref={getRef} className="truncate block">{text}</span>
    </div>
  )}
</SmartTooltip>
```

### RULE-SMTTP-03: Sử dụng `multipleLines` cho text dùng `line-clamp`

Single-line truncate dùng `scrollWidth` check. Multi-line clamp dùng `scrollHeight` check.

```tsx
// Single line
<SmartTooltip content={text}>
  {({ getRef }) => <p ref={getRef} className="truncate">{text}</p>}
</SmartTooltip>

// Multi-line clamp
<SmartTooltip content={text} multipleLines>
  {({ getRef }) => <p ref={getRef} className="line-clamp-2">{text}</p>}
</SmartTooltip>
```

### RULE-SMTTP-04: `children` là render prop — không bao giờ pass JSX trực tiếp

**Sai:**
```tsx
// ❌ Type error — children phải là function
<SmartTooltip content={text}>
  <span>{text}</span>
</SmartTooltip>
```

**Đúng:**
```tsx
// ✅
<SmartTooltip content={text}>
  {({ getRef }) => <span ref={getRef}>{text}</span>}
</SmartTooltip>
```
