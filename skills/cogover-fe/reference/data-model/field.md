---
title: Kiến thức về Field (DataField)
impact: HIGH
impactDescription: Field định nghĩa cấu trúc dữ liệu — chọn sai field type/metadata gây lỗi validation và rendering
tags: field, data-field, field-type, metadata, ui-kit
---

## Kiến thức về Field (DataField)

### Khái niệm

**Field (DataField)** định nghĩa cấu trúc của một trường dữ liệu trong object. Mỗi field có `slug`, `name`, `fieldType`, `fieldMetaData`, và các thuộc tính phân quyền/hiển thị.

### RULE-FIELD-01: Interface DataField

Các thuộc tính quan trọng:

| Property | Kiểu | Mô tả |
|---|---|---|
| `slug` | `string` | Định danh duy nhất, dùng làm key trong record |
| `name` | `string` | Tên hiển thị |
| `fieldType` | `DataTypes` | Loại field (xem RULE-FIELD-02) |
| `fieldMetaData` | `DataFieldMetaData` | Cấu hình chi tiết (format, limit, validation) |
| `required` | `number` | Bắt buộc (1) hay không (0) |
| `multiple` | `number` | Cho phép nhiều giá trị (1) hay không (0) |
| `options?` | `DataFieldOption[]` | Danh sách tùy chọn (cho `single_choice`, `multi_choices`, `radio_button`...) |
| `viewable`, `editable`, `creatable` | — | Quyền xem/sửa/tạo |
| `readOnly?` | `boolean \| number` | Chỉ đọc |
| `isStandard` | `number` | Field hệ thống (1) hay tùy chỉnh (0) |

### RULE-FIELD-02: Các loại Field Type (DataTypes)

Định nghĩa tại `ui-kit/src/apis/dataField/dataField.type.ts`.

| Nhóm | Field Types |
|------|-------------|
| Text | `short_text`, `long_text`, `label` |
| Lựa chọn | `single_choice`, `multi_choices`, `checkbox`, `boolean`, `radio_button`, `select_list` |
| Số | `numeric`, `decimal`, `percent`, `currency`, `auto_number` |
| Ngày giờ | `date`, `date_range`, `time`, `time_range`, `time_duration`, `date_time`, `date_time_range` |
| Liên kết | `lookup_parent`, `lookup_peer2peer`, `lookup_normal`, `embedded`, `reference`, `children`, `link` |
| Khác | `url`, `email`, `phone`, `file`, `regex`, `cascading`, `rating`, `formula`, `rollup_summary` |

### RULE-FIELD-03: fieldMetaData theo từng loại field

Mỗi field type có metadata riêng (vd: `ShortTextMetaData`, `NumberMetaData`, `DateMetaData`...) lưu trong `fieldMetaData`. Truy cập đúng kiểu metadata trước khi render UI hoặc validate.

### RULE-FIELD-04: Lấy thông tin field bằng useGetDataFieldBySlug

```typescript
import useGetDataFieldBySlug from "@stringeecom/ui-kit/hooks/useGetDataFieldBySlug";

const { fields, isLoading, object } = useGetDataFieldBySlug({
  objectSlug: "customer",
  dataFieldSlugs: ["name", "email", "phone"],
});

// fields: (DataField | null)[] — tương ứng thứ tự dataFieldSlugs truyền vào
```

Có thể dùng `dataFieldIds` thay cho `dataFieldSlugs`, và `objectId` thay cho `objectSlug`.

### RULE-FIELD-05: Field Lookup tự sinh Related List

Khi tạo field `reference` hoặc `lookup_normal` trong object A trỏ sang object B → object B tự sinh related list tương ứng. Xem reference `related-list`.
