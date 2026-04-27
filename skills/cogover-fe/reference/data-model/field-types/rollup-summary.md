---
title: Field Type — Tổng hợp (Roll-up Summary)
impact: HIGH
impactDescription: Chỉ tạo được trên parent object có dependency_lookup từ child; summary_type = SUM/COUNT/MIN/MAX/AVERAGE
tags: data-field, rollup, rollup_summary, RollUpSummaryMetaData, ui-kit
---

## Field Type — Tổng hợp (Roll-up Summary)

`fieldType = "rollup_summary"` (`DataTypes`). UI label: **"Tổng hợp (Roll-up Summary)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/rollup-summary`.

Cấu hình common: xem `field-types/common`. Phụ thuộc: `field-types/dependency-lookup`.

### RULE-FIELD-ROLLUP-01: Điều kiện để tạo

Object phải **có ít nhất 1 child** trỏ về qua `lookup_parent` (dependency lookup). Nếu chưa có → option "Tổng hợp" trong combobox tạo trường vẫn click được nhưng nút "Tiếp tục" **disabled**.

⚠️ Trong test environment với object child duy nhất (`test_shorttext_obj` → `user`), KHÔNG thể tạo rollup trên object child. API trả lỗi: `"Summarized Object is invalid"` (code 429) khi cố tạo rollup không hợp lệ.

### RULE-FIELD-ROLLUP-02: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Đối tượng tổng hợp | `object` | — (bắt buộc) | Combobox — chọn child object có dependency lookup tới object hiện tại |
| 2 | Trường tổng hợp | `field` | — (bắt buộc) | Combobox — chọn field number/numeric trên child để summary |
| 3 | Loại tổng hợp | `summaryType` | `"COUNT"` | Combobox 5 option (xem RULE-04) |
| 4 | Bộ lọc | `filterCriteria` / `rollupSummaryFilter` | — | Optional filter criteria để chỉ rollup record thoả filter |

### RULE-FIELD-ROLLUP-03: Input common bị ẩn / khoá

So với common (RULE-FIELD-COMMON-02):
- ❌ KHÔNG có Placeholder, isRequired, allowManualEdit, isMultipleValues, isUnique, defaultValue, hint
- ⚠️ Field luôn **read-only** (giống formula) — auto-compute từ child records

### RULE-FIELD-ROLLUP-04: `summaryType` — `AvgTypes` enum

```typescript
export enum AvgTypes {
    SUM = "SUM",
    COUNT = "COUNT",
    MIN = "MIN",
    MAX = "MAX",
    AVERAGE = "AVERAGE",
}
```

| Value | UI label | Hành vi |
|---|---|---|
| `"SUM"` | Tổng | Cộng tất cả giá trị field trên child records |
| `"COUNT"` | Đếm | Đếm số child records (không cần `field`) |
| `"MIN"` | Nhỏ nhất | Giá trị nhỏ nhất |
| `"MAX"` | Lớn nhất | Giá trị lớn nhất |
| `"AVERAGE"` | Trung bình | Giá trị trung bình |

`COUNT` không cần `field` trên child vì đếm số bản ghi; các loại khác đều cần `field` numeric trên child.

### RULE-FIELD-ROLLUP-05: Mapping form → `meta_data` (`RollUpSummaryMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:606-613`.

```typescript
export interface RollUpSummaryMetaData {
    object?: string;                        // ID của child object
    field?: string;                         // ID của field trên child cần summary
    field_slug?: string;                    // Slug của field (dùng cho display)
    summary_type?: AvgTypes;
    filter_criteria?: string | number;
    rollup_summary_filter?: string;         // JSON filter
}
```

| Form key | Đường dẫn |
|---|---|
| `object` (combobox child) | `meta_data.object` (child object id) |
| `field` (combobox field) | `meta_data.field` (field id) + `meta_data.field_slug` |
| `summaryType` | `meta_data.summary_type` |
| `filterCriteria` | `meta_data.filter_criteria` / `rollup_summary_filter` |

### RULE-FIELD-ROLLUP-06: Payload mẫu (theo schema)

Tạo rollup `total_orders` trên `Customer`, đếm số record `Order` (Order có dependency_lookup tới Customer):

```json
POST /api/v1/objects/field/create
{
  "type": "rollup_summary",
  "object_type_id": "<customer-id>",
  "workspace_id": "<ws-id>",
  "name": "Total Orders",
  "slug": "total_orders",
  "manual_modify_allow": false,
  "status": 1,
  "multiple": 0,
  "unique": 0,
  "meta_data": "{\"object\":\"<order-object-id>\",\"summary_type\":\"COUNT\"}"
}
```

Hoặc SUM giá trị `amount` của Order:

```json
{
  "meta_data": "{\"object\":\"<order-id>\",\"field\":\"<amount-field-id>\",\"field_slug\":\"amount\",\"summary_type\":\"SUM\"}"
}
```

### RULE-FIELD-ROLLUP-07: Lỗi BE thường gặp khi tạo

| Code `r` | `msg` | Nguyên nhân |
|---|---|---|
| `429` | `Summarized Object is invalid` | `meta_data.object` không phải child có dependency_lookup tới object hiện tại |
| `503` | `field not found` | `meta_data.field` không tồn tại trên child |
| `604` | `summary_type invalid` | `summary_type` không phải value enum hợp lệ |

### RULE-FIELD-ROLLUP-08: Khi child record thay đổi

Mỗi khi child record:
- **Tạo mới**: rollup parent re-compute (BE tự trigger)
- **Update field tham chiếu**: re-compute
- **Soft delete (status=0)**: re-compute (loại record khỏi summary)
- **Hard delete**: re-compute

Chú ý performance: nếu parent có > 10K children, mỗi update child sẽ re-compute → có thể chậm. BE thường debounce hoặc batch update.

### RULE-FIELD-ROLLUP-09: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`, `object`, `field`, `summary_type` (đổi summary_type cần migrate cache). Có thể sửa: `filter_criteria`, `rollup_summary_filter`, name/description.

Xoá child object hoặc field tham chiếu → rollup field bị invalid (BE tự xoá hoặc set null tuỳ config).
