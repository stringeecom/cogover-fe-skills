---
title: Cách sử dụng IconButton Component của ui-kit
impact: HIGH
impactDescription: Sử dụng IconButton sai (ví dụ: truyền startIcon hoặc thêm padding) gây ra layout bị hỏng hoặc thiếu tooltip
tags: icon-button, button, ui-kit
---

## Cách sử dụng IconButton Component của ui-kit

Đường dẫn component: `ui-kit/src/components/IconButton`

`IconButton` là wrapper mỏng trên `Button` để render một nút icon vuông không có padding. Props: `variant`, `size`, và tất cả `React.ButtonHTMLAttributes`. Hỗ trợ `ref` forwarding.

### RULE-ICON-01: Chỉ sử dụng `IconButton` trực tiếp khi không cần tooltip

Khi cần tooltip, sử dụng `Button` với `onlyIcon` thay thế.

```tsx
// ✅ Không cần tooltip → IconButton trực tiếp
<IconButton variant="text"><CloseIcon /></IconButton>

// ✅ Cần tooltip → Button onlyIcon
<Button onlyIcon startIcon={<CloseIcon />}>Đóng</Button>
```

### RULE-ICON-02: Truyền icon as `children`, không phải qua `startIcon`/`endIcon`

Khác với `Button`, `IconButton` không sử dụng props `startIcon`/`endIcon`.

**Sai:**
```tsx
// ❌ startIcon không phải là prop của IconButton
<IconButton startIcon={<EditIcon />} />
```

**Đúng:**
```tsx
// ✅
<IconButton><EditIcon /></IconButton>
```

### RULE-ICON-03: Không override `p-0` hoặc `min-w-[unset]`

`IconButton` enforce square sizing qua `p-0` và `min-w-[unset]`. Không override những cái này trong `className`.

## Tham chiếu kích thước

| size | width & height | font-size |
|------|----------------|-----------|
| `large` | 2.75rem (44px) | 18px |
| `medium` | 2rem (32px) | 16px |
| `small` | 1.75rem (28px) | 14px |
