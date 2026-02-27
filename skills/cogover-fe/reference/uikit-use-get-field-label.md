---
title: Cách sử dụng hook useGetFieldLabel trong ui-kit
impact: MEDIUM
impactDescription: Hook lấy label hiển thị của field từ slug — tự fetch và tìm thủ công gây duplicate API calls
tags: hook, field, label, object, slug, ui-kit
---

## Cách sử dụng hook useGetFieldLabel trong ui-kit

Đường dẫn hook: `ui-kit/src/hooks/useGetFieldLabel`

Hook chuyển đổi mảng field slugs thành mảng tên hiển thị (labels) tương ứng.

### Props

| Prop | Kiểu | Mặc định | Mô tả |
| --- | --- | --- | --- |
| `objectSlug` | `string` | — | Slug của object (bắt buộc) |
| `fieldSlugs` | `string[]` | — | Mảng field slugs cần lấy label (bắt buộc) |
| `hasTranslate` | `boolean` | `true` | Có lấy tên đã dịch không |

### Giá trị trả về

| Giá trị | Kiểu | Mô tả |
| --- | --- | --- |
| `labels` | `string[]` | Mảng tên field tương ứng (chuỗi rỗng nếu không tìm thấy) |

### RULE-GFL-01: Luôn truyền đủ `objectSlug` và `fieldSlugs` — đây là 2 props bắt buộc

```tsx
const labels = useGetFieldLabel({
    objectSlug: "contacts",
    fieldSlugs: ["email", "phone", "name"],
});
// labels = ["Email", "Số điện thoại", "Họ tên"]
```

### RULE-GFL-02: Không tự fetch object và tìm field name thủ công — hook đã tích hợp `useQueryObjectDetailBySlug`

```tsx
// ❌ Tự fetch
const { data: object } = useQueryObjectDetailBySlug({ slug: "contacts" });
const label = object?.fields.find(f => f.slug === "email")?.name ?? "";

// ✅ Sử dụng hook
const [emailLabel] = useGetFieldLabel({
    objectSlug: "contacts",
    fieldSlugs: ["email"],
});
```

### RULE-GFL-03: Kết quả giữ nguyên thứ tự và kích thước mảng đầu vào — chuỗi rỗng cho slug không tìm thấy

```tsx
const labels = useGetFieldLabel({
    objectSlug: "contacts",
    fieldSlugs: ["email", "nonexistent"],
});
// labels = ["Email", ""]
```

### RULE-GFL-04: Truyền `hasTranslate={false}` khi cần tên gốc (chưa dịch) của field

```tsx
// ✅ Lấy tên đã dịch (mặc định)
const labels = useGetFieldLabel({ objectSlug: "contacts", fieldSlugs: ["email"] });

// ✅ Lấy tên gốc
const labels = useGetFieldLabel({ objectSlug: "contacts", fieldSlugs: ["email"], hasTranslate: false });
```
