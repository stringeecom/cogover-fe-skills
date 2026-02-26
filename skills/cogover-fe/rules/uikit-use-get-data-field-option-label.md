---
title: Cách sử dụng hook useGetDataFieldOptionLabel trong ui-kit
impact: MEDIUM
impactDescription: Hook lấy label hiển thị của option trong field — tự tìm kiếm thủ công gây duplicate logic và dữ liệu không đồng bộ
tags: hook, data-field, option, label, object, ui-kit
---

## Cách sử dụng hook useGetDataFieldOptionLabel trong ui-kit

Đường dẫn hook: `ui-kit/src/hooks/useGetDataFieldOptionLabel`

Hook cung cấp hàm tiện ích để lấy label hiển thị của một option cụ thể trong field options.

### Props

| Prop | Kiểu | Mô tả |
| --- | --- | --- |
| `objectSlug` | `string` | Slug của object chứa field (bắt buộc) |

### Giá trị trả về

| Giá trị | Kiểu | Mô tả |
| --- | --- | --- |
| `getOptionLabel` | `(params: { fieldSlug: string; optionSlug: string }) => string` | Hàm lấy label của option |

### RULE-GDFOL-01: Luôn truyền `objectSlug` — đây là prop bắt buộc duy nhất

```tsx
const { getOptionLabel } = useGetDataFieldOptionLabel({ objectSlug: "contacts" });
```

### RULE-GDFOL-02: Sử dụng `getOptionLabel` thay vì tự tìm kiếm option trong fields

```tsx
// ❌ Tự tìm kiếm
const field = object?.fields.find(f => f.slug === "status");
const option = field?.options?.find(o => o.slug === "active");
const label = option?.value ?? "";

// ✅ Sử dụng hook
const { getOptionLabel } = useGetDataFieldOptionLabel({ objectSlug: "contacts" });
const label = getOptionLabel({ fieldSlug: "status", optionSlug: "active" });
```

### RULE-GDFOL-03: Hàm trả về chuỗi rỗng khi không tìm thấy field hoặc option — không cần kiểm tra null

```tsx
// ✅ An toàn, trả về "" nếu không tìm thấy
const label = getOptionLabel({ fieldSlug: "nonexistent", optionSlug: "any" });
// label = ""
```

### RULE-GDFOL-04: Callback `getOptionLabel` đã được memoize — an toàn khi dùng trong dependency array

```tsx
// ✅ Dùng trong useMemo, useEffect
const displayText = useMemo(() => {
    return getOptionLabel({ fieldSlug: "status", optionSlug: record.status });
}, [getOptionLabel, record.status]);
```
