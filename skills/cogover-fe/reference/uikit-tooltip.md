---
title: Cách sử dụng Tooltip Component của ui-kit
impact: HIGH
impactDescription: Sử dụng Tooltip sai gây ra vấn đề UX trên mobile, lỗi React render, hoặc duplicate DOM không cần thiết
tags: tooltip, overlay, mobile, ui-kit
---

## Cách sử dụng Tooltip Component của ui-kit

Đường dẫn component: `ui-kit/src/components/Tooltip`
Xây dựng trên: `@tippyjs/react`

Props: `disabledOnTouchedDevice` (mặc định `true`), cộng tất cả `TippyProps` (`content`, `placement`, `disabled`, `delay`, `trigger`, `children`, v.v.).

### RULE-TTP-01: `children` phải là một React element duy nhất

Tippy yêu cầu một React element duy nhất làm trigger — không phải text, fragment, hoặc array.

**Sai:**
```tsx
// ❌ Text node làm children
<Tooltip content="Info">hover me</Tooltip>

// ❌ Fragment
<Tooltip content="Info"><><span>A</span><span>B</span></></Tooltip>
```

**Đúng:**
```tsx
// ✅
<Tooltip content="Info"><span>hover me</span></Tooltip>
```

### RULE-TTP-02: Tooltip tự động bị tắt trên touch/mobile devices

`disabledOnTouchedDevice` mặc định là `true`. Tooltip tự động ẩn trên touch devices. Chỉ set `false` khi cần rõ ràng trên mobile.

```tsx
// ✅ Enable trên mobile/touch
<Tooltip content="Info" disabledOnTouchedDevice={false}>
  <button>tap me</button>
</Tooltip>
```

### RULE-TTP-03: Sử dụng prop `disabled` — không bao giờ conditionally render `<Tooltip>`

**Sai:**
```tsx
// ❌ Gây ra unmount/remount không cần thiết
{shouldShow ? (
  <Tooltip content="Info"><button>...</button></Tooltip>
) : (
  <button>...</button>
)}
```

**Đúng:**
```tsx
// ✅
<Tooltip content="Info" disabled={!shouldShow}>
  <button>...</button>
</Tooltip>
```

### RULE-TTP-04: Ưu tiên `SmartTooltip` cho text có khả năng bị truncate

Khi tooltip chỉ nên xuất hiện khi text bị cắt, sử dụng `SmartTooltip` — nó tự động detect overflow.

### RULE-TTP-05: Không override core tooltip styles

Background `#484848`, white text, và arrow color đã được pre-configured. Không override những cái này qua `className` trừ khi thực sự cần thiết.
