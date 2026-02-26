---
title: Cách sử dụng Button Component của ui-kit
impact: HIGH
impactDescription: Sử dụng Button sai dẫn đến việc wrap dư thừa, trạng thái loading không nhất quán, và đặt icon sai vị trí
tags: button, icon, loading, onlyIcon, ui-kit
---

## Cách sử dụng Button Component của ui-kit

Đường dẫn component: `ui-kit/src/components/Button`

Props: `variant`, `size`, `onlyIcon`, `startIcon`, `endIcon`, `fullWidth`, `loading`, `disabledTooltip`. Hỗ trợ tất cả `React.ButtonHTMLAttributes` và `ref` forwarding.

### RULE-BTN-01: Sử dụng `Button onlyIcon` thay vì wrap `IconButton` với `Tooltip`

Khi một button chỉ hiển thị icon và cần tooltip, sử dụng `Button` với `onlyIcon` (children sẽ trở thành text của tooltip). KHÔNG wrap `IconButton` với `Tooltip` một cách thủ công.

**Sai:**
```tsx
// ❌ Wrap thủ công là dư thừa — Button onlyIcon đã làm điều này internally
<Tooltip content="Xóa">
  <IconButton><DeleteIcon /></IconButton>
</Tooltip>
```

**Đúng:**
```tsx
// ✅ children được dùng làm tooltip text tự động
<Button onlyIcon startIcon={<DeleteIcon />}>Xóa</Button>
```

### RULE-BTN-02: Sử dụng prop `loading` thay vì spinner thủ công + disabled

**Sai:**
```tsx
// ❌ Duplicate những gì mà prop loading đã xử lý
<Button disabled={isSubmitting}>
  {isSubmitting ? <Spinner /> : 'Save'}
</Button>
```

**Đúng:**
```tsx
// ✅ loading tự động xử lý disabled state và spinner
<Button loading={isSubmitting}>Save</Button>
```

### RULE-BTN-03: Truyền icons qua `startIcon`/`endIcon`, không phải children

**Sai:**
```tsx
// ❌ Icon lẫn trong children
<Button><SaveIcon /> Save</Button>
```

**Đúng:**
```tsx
// ✅
<Button startIcon={<SaveIcon />}>Save</Button>
<Button endIcon={<ArrowIcon />}>Next</Button>
```

### RULE-BTN-04: Sử dụng `disabledTooltip` khi không muốn tooltip trên `onlyIcon`

```tsx
// ✅ Tắt auto-tooltip khi context đã làm label trở nên dư thừa
<Button onlyIcon disabledTooltip startIcon={<DeleteIcon />}>Xóa</Button>
```

## Tham chiếu kích thước

| size | height | padding-x |
|------|--------|-----------|
| `large` | 2.75rem | 1rem |
| `medium` | 2rem | 0.75rem |
| `small` | 1.75rem | 0.75rem |
