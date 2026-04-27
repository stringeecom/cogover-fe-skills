---
title: Field Type — Nhãn (Label)
impact: MEDIUM
impactDescription: Label gần giống short_text nhưng không có display_type; dùng cho phân loại / gắn nhãn
tags: data-field, label, LabelsMetaData, ui-kit
---

## Field Type — Nhãn (Label)

`fieldType = "label"` (`DataTypes`). UI label: **"Nhãn (Label)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/label`.

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-LABEL-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Số lượng ký tự — Tối thiểu | `minLength` | `0` | NumberInput |
| 2 | Số lượng ký tự — Tối đa | `maxLength` | `255` | NumberInput, max 255 |

Khi `isMultipleValues = true`: hiện thêm 2 input `multiple_limit.min` / `multiple_limit.max`.

### RULE-FIELD-LABEL-02: Input common

So với common (RULE-FIELD-COMMON-02): **đầy đủ 12 input** (giống short-text), nhưng **không có** combobox "Kiểu hiển thị". KHÔNG có "Giá trị mặc định" (`defaultValue`) — verify từ form.

### RULE-FIELD-LABEL-03: Mapping form → `meta_data` (`LabelsMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:231-241`.

```typescript
export interface LabelsMetaData {
    character_limit?: { min?: number; max?: number };
    multiple_limit?: { min?: number; max?: number };
    unique_rule?: number;
}
```

| Form key | Đường dẫn trong `meta_data` |
|---|---|
| `minLength` / `maxLength` | `character_limit.min/max` |
| `multiple_limit.min/max` | `multiple_limit.min/max` |

### RULE-FIELD-LABEL-04: Payload thực tế (verify từ API)

Tạo "Test Label Field" với defaults:

```json
{
  "fieldType": "label",
  "fieldMetaData": "{\"character_limit\":{\"min\":0,\"max\":255},\"multiple_limit\":{\"min\":0,\"max\":30}}"
}
```

### RULE-FIELD-LABEL-05: Khác biệt với `short_text`

| Khía cạnh | `short_text` | `label` |
|---|---|---|
| `display_type` (1-line / multi-line) | Có | Không (luôn render 1 dòng) |
| Render record | Plain input | Tag/Pill UI (badge) |
| Use case | Tên, mã, code... | Phân loại, gán nhãn (vd "VIP", "New", "Pending") |
| Default `multiple_limit.min` | 1 | 0 |

### RULE-FIELD-LABEL-06: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`. Có thể sửa: `character_limit`, `multiple_limit`, các thuộc tính common.
