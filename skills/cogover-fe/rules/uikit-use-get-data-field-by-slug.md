---
title: Cách sử dụng hook useGetDataFieldBySlug trong ui-kit
impact: HIGH
impactDescription: Hook lấy metadata field từ object — tự fetch hoặc sử dụng sai gây duplicate API calls và dữ liệu không nhất quán
tags: hook, data-field, object, slug, api, ui-kit
---

## Cách sử dụng hook useGetDataFieldBySlug trong ui-kit

Đường dẫn hook: `ui-kit/src/hooks/useGetDataFieldBySlug`

Hook lấy thông tin chi tiết của các data fields từ object definition theo slug hoặc ID.

### Props

| Prop | Kiểu | Mô tả |
| --- | --- | --- |
| `dataFieldSlugs` | `string[]` | Mảng field slugs cần lấy (ưu tiên hơn dataFieldIds) |
| `dataFieldIds` | `string[]` | Mảng field IDs cần lấy (fallback) |
| `objectSlug` | `string` | Slug của object |
| `objectId` | `string` | ID của object (fallback nếu không có objectSlug) |

### Giá trị trả về

| Giá trị | Kiểu | Mô tả |
| --- | --- | --- |
| `fields` | `(DataField \| null)[]` | Mảng fields tương ứng (null nếu không tìm thấy) |
| `isLoading` | `boolean` | Trạng thái loading |
| `object` | `ObjectListResponse \| undefined` | Dữ liệu object đầy đủ |

### RULE-GDFBS-01: Ưu tiên truyền `objectSlug` — chỉ dùng `objectId` khi không có slug

```tsx
// ✅ Ưu tiên objectSlug
const { fields } = useGetDataFieldBySlug({
    objectSlug: "contacts",
    dataFieldSlugs: ["email", "phone"],
});

// ✅ Fallback sang objectId
const { fields } = useGetDataFieldBySlug({
    objectId: "abc123",
    dataFieldIds: ["field1", "field2"],
});
```

### RULE-GDFBS-02: Ưu tiên `dataFieldSlugs` — hook sẽ bỏ qua `dataFieldIds` nếu `dataFieldSlugs` được truyền

```tsx
// ✅ Dùng slugs
const { fields } = useGetDataFieldBySlug({
    objectSlug: "contacts",
    dataFieldSlugs: ["email", "phone"],
});
```

### RULE-GDFBS-03: Kết quả trả về giữ nguyên thứ tự và kích thước mảng đầu vào — phần tử null nếu không tìm thấy

```tsx
const { fields } = useGetDataFieldBySlug({
    objectSlug: "contacts",
    dataFieldSlugs: ["email", "nonexistent", "phone"],
});
// fields = [DataField, null, DataField] — giữ nguyên thứ tự
```

### RULE-GDFBS-04: Không tự fetch object và tìm field thủ công — hook đã tích hợp `useQueryObjectDetailBySlug`

```tsx
// ❌ Tự fetch và tìm
const { data: object } = useQueryObjectDetailBySlug({ slug: "contacts" });
const emailField = object?.fields.find(f => f.slug === "email");

// ✅ Sử dụng hook
const { fields } = useGetDataFieldBySlug({
    objectSlug: "contacts",
    dataFieldSlugs: ["email"],
});
```

### RULE-GDFBS-05: Kiểm tra `isLoading` trước khi sử dụng `fields` — tránh render với mảng rỗng

```tsx
const { fields, isLoading } = useGetDataFieldBySlug({ ... });

if (isLoading) return <Skeleton />;
```
