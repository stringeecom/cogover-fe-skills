---
title: Field Type — Tổng hợp (Roll-up Summary)
impact: HIGH
impactDescription: Chỉ tạo được trên parent object có dependency_lookup từ child (child phải có field type=reference với link_field=id). summary_type=COUNT không cần field, các loại khác cần numeric field
tags: data-field, rollup, rollup_summary, RollUpSummaryMetaData, ui-kit
---

## Field Type — Tổng hợp (Roll-up Summary)

`fieldType = "rollup_summary"` (`DataTypes`). UI label: **"Tổng hợp (Roll-up Summary)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/rollup` (slug `rollup`, KHÔNG phải `rollup-summary`).

Cấu hình common: xem `field-types/common`. Phụ thuộc: `field-types/dependency-lookup`.

### RULE-FIELD-ROLLUP-01: Điều kiện để tạo

**Object hiện tại phải là parent của ít nhất 1 child** thông qua **dependency lookup**. Ở Cogover, dependency lookup ≡ field có `fieldType: "reference"` + `meta_data.link_field: "id"` trên child object trỏ về parent (xem `field-types/dependency-lookup`).

Khi mở form rollup-summary, FE gọi `GET /api/v1/objects/object/get-dependants/{parentObjectId}` để lấy danh sách children. Nếu rỗng:
- Card "Tổng hợp" trên màn chọn data type vẫn click được nhưng nút **"Tiếp tục" disabled**
- Hoặc khi vào form, combobox "Đối tượng được tổng hợp" không có option nào

### RULE-FIELD-ROLLUP-02: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Đối tượng được tổng hợp | `summaryObject` | — (bắt buộc) | Combobox — chỉ list các child object qua `get-dependants` |
| 2 | Loại tổng hợp | `summaryType` | (chưa chọn — bắt buộc) | Combobox 5 option (xem RULE-04) |
| 3 | Trường cần tổng hợp | `summaryField` | — (bắt buộc khi summaryType ≠ COUNT) | Combobox — list numeric/decimal/percent/currency field trên child. **Tự ẩn khi `summaryType = COUNT`** |
| 4 | Tiêu chí lọc để tổng hợp | (radio + filter builder) | "Tất cả các bản ghi liên quan" | Radio 2 option: "Tất cả..." hoặc "Chỉ tổng hợp dữ liệu từ những bản ghi đáp ứng điều kiện lọc" |

### RULE-FIELD-ROLLUP-03: Input common bị ẩn / khoá

So với common (RULE-FIELD-COMMON-02):
- ❌ KHÔNG có Placeholder, `isRequired`, `quickSearch`, `isMultipleValues`, `isUnique`, `defaultValue`, hint
- ✅ CÓ "Cho phép tạo, sửa thủ công" — payload gửi `manual_modify_allow: true` (default)
- ⚠️ Field luôn **read-only** trên record (auto-compute từ child)

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

| Value | UI label | Mô tả ngắn (UI) | Hành vi |
|---|---|---|---|
| `"SUM"` | SUM | Tính Tổng | Cộng tất cả giá trị field trên child records |
| `"COUNT"` | COUNT | Đếm số lượng bản ghi | Đếm số child records — **KHÔNG cần field** |
| `"MIN"` | MIN | Giá trị nhỏ nhất | Giá trị nhỏ nhất |
| `"MAX"` | MAX | Giá trị lớn nhất | Giá trị lớn nhất |
| `"AVERAGE"` | AVERAGE | Giá trị trung bình | Giá trị trung bình |

⚠️ Khi user chọn `COUNT` ở combobox, **field "Trường cần tổng hợp" tự ẩn khỏi UI** (không phải disabled — DOM bị remove). Form vẫn submit được mà không cần chọn field.

### RULE-FIELD-ROLLUP-05: Mapping form → `meta_data` (`RollUpSummaryMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts` (interface `RollUpSummaryMetaData`).

```typescript
export interface RollUpSummaryMetaData {
    object?: string;                        // ID của child object
    field?: string;                         // ID/slug của field trên child cần summary (rỗng cho COUNT)
    field_slug?: string;                    // Slug của field (rỗng cho COUNT)
    summary_type?: AvgTypes;
    filter_criteria?: string | number | null; // null khi không filter
}
```

| Form key | Đường dẫn |
|---|---|
| `summaryObject` (combobox) | `meta_data.object` (child object id) |
| `summaryType` | `meta_data.summary_type` |
| `summaryField` | `meta_data.field` + `meta_data.field_slug` (rỗng `""` nếu COUNT) |
| Radio "Tất cả" | `meta_data.filter_criteria: null` + `rollup_summary_filter` rỗng |
| Radio "Chỉ tổng hợp..." + filter builder | `rollup_summary_filter: { logicType, logic, conditions, sortFields }` (top-level) |

⚠️ **`rollup_summary_filter` ở top-level body**, KHÔNG nằm trong `meta_data`. Khác với type def cũ ghi `rollup_summary_filter?: string` trong meta_data.

### RULE-FIELD-ROLLUP-06: Payload thực tế (verify từ API)

Tạo `Total Test Records` trên `Object 1`, COUNT records của `Test`:

```json
POST /api/v1/objects/field/create
{
  "type": "rollup_summary",
  "object_type_id": "OT2THCQE8KMS4",
  "workspace_id": "WSsgfBFZxLurz",
  "name": "Total Test Records",
  "slug": "total_test_records",
  "tool_tip": "",
  "status": 1,
  "description": "",
  "hint_text": "",
  "required": 0,
  "manual_modify_allow": true,
  "meta_data": "{\"object\":\"OTTPOTQELKM83\",\"field\":\"\",\"field_slug\":\"\",\"summary_type\":\"COUNT\",\"filter_criteria\":null}",
  "rollup_summary_filter": {
    "logicType": "AND",
    "logic": "",
    "conditions": [],
    "sortFields": []
  }
}
```

**Phát hiện**:
- `meta_data.field = ""` và `meta_data.field_slug = ""` cho COUNT — gửi empty string, KHÔNG omit hay null
- `meta_data.filter_criteria = null`
- `rollup_summary_filter` luôn gửi (kể cả khi không filter) với cấu trúc rỗng `{logicType:"AND", logic:"", conditions:[], sortFields:[]}`
- KHÔNG có `multiple`, `unique`, `quickSearch` trong body
- Response: `{ r: 0, msg: "OK", data: [{ id, slug, options: [] }] }`

Cho SUM/MIN/MAX/AVERAGE (theo schema):

```json
{
  "meta_data": "{\"object\":\"<child-id>\",\"field\":\"<numeric-field-id>\",\"field_slug\":\"amount\",\"summary_type\":\"SUM\",\"filter_criteria\":null}"
}
```

### RULE-FIELD-ROLLUP-07: Lỗi BE thường gặp

| Code `r` | `msg` | Nguyên nhân |
|---|---|---|
| `429` | `Summarized Object is invalid` | `meta_data.object` không phải child object có dependency tới `object_type_id` (parent). Thường do field "dependency" trên child không phải `type: reference` |
| `503` | `field not found` | `meta_data.field` không tồn tại trên child |
| `604` | `summary_type invalid` | `summary_type` không phải value enum hợp lệ |

### RULE-FIELD-ROLLUP-08: Khi child record thay đổi

Mỗi khi child record:
- **Tạo mới**: rollup parent re-compute (BE tự trigger)
- **Update field tham chiếu**: re-compute
- **Soft delete (status=0)**: re-compute (loại record khỏi summary)
- **Hard delete**: re-compute
- **Đổi parent (cập nhật `link_field`)**: re-compute cả parent cũ + parent mới

Chú ý performance: nếu parent có > 10K children, mỗi update child re-compute có thể chậm. BE thường debounce hoặc batch update.

### RULE-FIELD-ROLLUP-09: Filter `rollup_summary_filter`

Khi user chọn radio "Chỉ tổng hợp dữ liệu từ những bản ghi đáp ứng điều kiện lọc" + thêm conditions, payload có:

```json
"rollup_summary_filter": {
  "logicType": "AND",
  "logic": "1 AND 2",
  "conditions": [
    { "field": "amount", "op": ">", "params": 100 },
    { "field": "status", "op": "=", "params": "active" }
  ],
  "sortFields": []
}
```

`filter_criteria` trong `meta_data` lưu **id filter** (FIxxx) khi filter được lưu thành filter chia sẻ; rỗng/null khi inline filter.

### RULE-FIELD-ROLLUP-10: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`, `meta_data.object`, `meta_data.field`, `meta_data.summary_type`. Có thể sửa: `rollup_summary_filter`, name, description, status.

Xoá child object hoặc field tham chiếu → rollup field bị invalid (BE tự xoá hoặc set null tuỳ config).

### RULE-FIELD-ROLLUP-11: Workspace lookup test reference (long02)

Workspace `WSsgfBFZxLurz` trên `long02.release.cogover.net` có sẵn:
- Object `test` (id `OTTPOTQELKM83`) — có 28 distinct fieldType + 1 field `lookup_dependency` (`fieldType: "reference"`, `link_field: "id"`) trỏ về `Object 1`
- Object `Object 1` (id `OT2THCQE8KMS4`, slug `object_1`) — parent → có thể tạo rollup_summary tổng hợp records của `test`

Dùng làm reference khi cần verify rollup behavior.
