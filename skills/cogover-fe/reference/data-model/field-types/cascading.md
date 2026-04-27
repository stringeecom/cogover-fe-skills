---
title: Field Type — Cây thư mục (Cascading)
impact: HIGH
impactDescription: format quyết định mode chọn (single-leaf/single-any/multi-leaf/multi-any); options array lưu tree nodes (parent_id quan hệ cha-con)
tags: data-field, cascading, TreeMetaData, ui-kit
---

## Field Type — Cây thư mục (Cascading)

`fieldType = "cascading"` (`DataTypes`). UI label: **"Cây thư mục (Cascading)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/cascading`.

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-CASCADING-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Định dạng cây thư mục | `formatTreeType` | (chưa chọn — bắt buộc) | Combobox 4 option (xem RULE-02) |
| 2 | Tạo cây thư mục | `treeOptions` | `[]` (bắt buộc ≥ 1 node) | Tree builder UI cho phép thêm folder/sub-folder |
| 3 | Tính duy nhất | `isUnique` | `false` | (giống common) |
| 4 | Giá trị mặc định | `defaultValue` | `null` | Combobox chọn từ tree đã tạo |

### RULE-FIELD-CASCADING-02: Combobox `formatTreeType` — 4 mode

| UI label | Value (`format`) | Hành vi |
|---|---|---|
| Chỉ chọn 1 nhánh con | `1` | Chỉ chọn được **leaf node** (node không có con), single value |
| Chỉ chọn 1 nhánh/ thư mục bất kỳ | `2` | Chọn được node bất kỳ (kể cả folder cha), single value |
| Chọn nhiều nhánh con | `3` | Multi-select chỉ leaf nodes |
| Chọn nhiều nhánh/ thư mục bất kỳ | `4` | Multi-select node bất kỳ |

### RULE-FIELD-CASCADING-03: Input common bị ẩn

So với common (RULE-FIELD-COMMON-02):
- ❌ KHÔNG có "Cho phép sử dụng để Tìm kiếm nhanh" (`quickSearch`)
- ❌ KHÔNG có "Cho phép trường có nhiều giá trị" (`isMultipleValues`) — multi/single quyết định bởi `formatTreeType`

### RULE-FIELD-CASCADING-04: Mapping form → `meta_data` (`TreeMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:443-450`.

```typescript
export interface TreeMetaData {
    multiple_limit?: { min?: number; max?: number };
    unique_rule?: number;
    format: number;        // 1-4 theo RULE-02
}

export interface OptionTreeFolder {
    id?: string | number | null;
    value: string;
    parent_id?: string | null;
}
```

| Form key | Đường dẫn trong `meta_data` |
|---|---|
| `formatTreeType` | `format` (1-4) |
| `multiple_limit` | `multiple_limit.min/max` (auto theo format) |

### RULE-FIELD-CASCADING-05: Tree nodes lưu trong `options` (TOP-LEVEL)

Mỗi node là 1 entry trong `options[]` array — KHÔNG nằm trong `fieldMetaData`. Quan hệ cha-con qua `parent_id`:

```json
"options": [
  { "id": "OPT-1", "value": "Hà Nội",        "parent_id": null,    "sort": 1 },
  { "id": "OPT-2", "value": "Cầu Giấy",      "parent_id": "OPT-1", "sort": 1 },
  { "id": "OPT-3", "value": "Đống Đa",       "parent_id": "OPT-1", "sort": 2 },
  { "id": "OPT-4", "value": "TP.HCM",        "parent_id": null,    "sort": 2 },
  { "id": "OPT-5", "value": "Quận 1",        "parent_id": "OPT-4", "sort": 1 }
]
```

`parent_id = null` = root node (folder cấp 1).

### RULE-FIELD-CASCADING-06: Payload thực tế (verify từ API)

Tạo "Test Cascading Field" với format `1` (chỉ leaf, single) và 1 node "Folder A":

```json
{
  "fieldType": "cascading",
  "fieldMetaData": "{\"multiple_limit\":{\"min\":0,\"max\":1},\"format\":1}",
  "options": [{
    "id": "OPTPOTQETKM9A",
    "value": "Folder A",
    "slug": "Folder_A",
    "sort": 1,
    "status": 1,
    "isDefault": 0
  }]
}
```

`multiple_limit.max = 1` (auto từ format=1 single).

### RULE-FIELD-CASCADING-07: Slug node tự gen từ value

`slug` = value với space → `_` (vd "Folder A" → `Folder_A`). Nếu trùng slug giữa các node cùng cha → BE thêm số (vd `Folder_A`, `Folder_A_2`).

### RULE-FIELD-CASCADING-08: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`, `format` (nếu đã có record dùng — verify). Có thể sửa: thêm/sửa/xoá tree nodes, defaultValue. Đổi `format` từ single → multi thường cho phép, ngược lại không (data loss).
