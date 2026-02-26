---
title: Cách sử dụng hook useGetLayout trong ui-kit
impact: HIGH
impactDescription: Hook quản lý form layout phức tạp — sử dụng sai gây render sai layout, mất fields, hoặc lỗi permissions
tags: hook, layout, form-builder, object, standard-layout, ui-kit
---

## Cách sử dụng hook useGetLayout trong ui-kit

Đường dẫn hook: `ui-kit/src/hooks/useGetLayout`

Hook toàn diện cho quản lý form layout với hỗ trợ standard/custom layouts, lọc fields, chế độ view/add, và chuyển đổi layout.

### Props

| Prop | Kiểu | Mặc định | Mô tả |
| --- | --- | --- | --- |
| `functionLabel` | `"ADD" \| "VIEW"` | — | Chế độ form: tạo mới hoặc xem chi tiết (bắt buộc) |
| `objectSlug` | `string` | — | Slug của object (bắt buộc) |
| `saveToSearchParams` | `boolean` | `false` | Lưu layout ID đang chọn vào URL |
| `layoutId` | `string` | — | ID layout cụ thể |
| `hiddenFieldSlugs` | `string[]` | — | Mảng field slugs cần ẩn |
| `readOnlyFieldSlugs` | `string[]` | — | Mảng field slugs chỉ đọc |
| `prioritizeStandardLayout` | `boolean` | `false` | Ưu tiên standard layout |
| `allowReadOnlyField` | `boolean` | `false` | Hiển thị field chỉ đọc trong create mode |

### Giá trị trả về

| Giá trị | Kiểu | Mô tả |
| --- | --- | --- |
| `layout` | `FormBuilderLayoutRow[]` | Cấu trúc layout đã xử lý |
| `listLayouts` | `GetLayoutMetaItem[]` | Danh sách layouts có sẵn |
| `activeLayoutId` | `string \| null` | ID layout đang chọn |
| `activeLayoutSlug` | `string \| null` | Slug layout đang chọn |
| `setActiveLayoutId` | `(layoutId: string) => void` | Hàm chuyển layout |
| `isLoading` | `boolean` | Trạng thái loading |
| `isStandardLayout` | `boolean` | Đang dùng standard layout |
| `layoutResponse` | `LayoutResponse \| null` | Dữ liệu layout gốc từ API |

### RULE-GL-01: Luôn truyền đủ `functionLabel` và `objectSlug` — đây là 2 props bắt buộc

```tsx
// ✅ Create mode
const layoutData = useGetLayout({ functionLabel: "ADD", objectSlug: "contacts" });

// ✅ View/Edit mode
const layoutData = useGetLayout({ functionLabel: "VIEW", objectSlug: "contacts" });
```

### RULE-GL-02: Sử dụng `functionLabel="ADD"` cho form tạo mới, `"VIEW"` cho form xem/sửa — không nhầm lẫn

ADD mode lọc bớt các field đặc biệt (display_box, related_list, tracking_history...) mà chỉ có ý nghĩa khi đã có record.

### RULE-GL-03: Sử dụng `hiddenFieldSlugs` để ẩn fields — không tự lọc layout trả về

```tsx
// ❌ Tự lọc
const layoutData = useGetLayout({ functionLabel: "VIEW", objectSlug: "contacts" });
const filteredLayout = layoutData.layout.filter(/* manual filtering */);

// ✅ Truyền vào hook
const layoutData = useGetLayout({
    functionLabel: "VIEW",
    objectSlug: "contacts",
    hiddenFieldSlugs: ["internal_notes", "secret_field"],
});
```

### RULE-GL-04: Sử dụng `readOnlyFieldSlugs` để đánh dấu fields chỉ đọc — không tự set readOnly trên từng component

```tsx
// ✅ Hook sẽ tự set readOnly cho các fields này trong layout
const layoutData = useGetLayout({
    functionLabel: "VIEW",
    objectSlug: "contacts",
    readOnlyFieldSlugs: ["created_date", "created_by"],
});
```

### RULE-GL-05: Kết hợp với `LayoutSelectButton` để cho phép người dùng chuyển layout

```tsx
const { listLayouts, activeLayoutId, setActiveLayoutId, layout } = useGetLayout({
    functionLabel: "VIEW",
    objectSlug: "contacts",
    saveToSearchParams: true,
});

<LayoutSelectButton
    listLayouts={listLayouts}
    activeLayoutId={activeLayoutId}
    onChangeLayout={setActiveLayoutId}
/>
```

### RULE-GL-06: Kiểm tra `isLoading` trước khi render FormBuilder — tránh render với layout rỗng

```tsx
const { layout, isLoading } = useGetLayout({ ... });

if (isLoading) return <Skeleton />;

return <FormBuilder layout={layout} />;
```

### RULE-GL-07: Sử dụng `layoutId` khi cần hiển thị layout cụ thể — bỏ qua để hook tự chọn layout mặc định

```tsx
// ✅ Chỉ định layout cụ thể
const layoutData = useGetLayout({
    functionLabel: "VIEW",
    objectSlug: "contacts",
    layoutId: "specific-layout-id",
});
```
