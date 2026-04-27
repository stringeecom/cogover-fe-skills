---
title: Field Type — Boolean
impact: HIGH
impactDescription: Sai cấu hình true_value/false_value khiến record hiển thị "Yes/No" thô thay vì label localize
tags: data-field, boolean, BooleanMetaData, ui-kit
---

## Field Type — Boolean

`fieldType = "boolean"` (`DataTypes`). UI label: **"Boolean"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/boolean`.

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-BOOLEAN-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Mặc định — Chọn | `defaultValue` | `false` | Checkbox đơn — bật = default `true`, tắt = default `false` |
| 2 | Thiết lập giá trị hiển thị — Chọn (true_value) | `trueValue` | `"Có"` | Textbox, **bắt buộc nhập** — nhãn hiển thị khi value = true |
| 3 | Thiết lập giá trị hiển thị — Không chọn (false_value) | `falseValue` | `"Không"` | Textbox, **bắt buộc nhập** — nhãn hiển thị khi value = false |

### RULE-FIELD-BOOLEAN-02: Input common bị ẩn / khoá

So với common (RULE-FIELD-COMMON-02):
- ❌ KHÔNG có Placeholder (`hint`) — boolean không phải input text
- ❌ KHÔNG có "Cho phép sử dụng để Tìm kiếm nhanh" (`quickSearch`) — boolean không phải search target
- ❌ KHÔNG có "Cho phép trường có nhiều giá trị" (`isMultipleValues`) — boolean luôn single
- ❌ KHÔNG có "Tính duy nhất" (`isUnique`) — boolean trùng nhau là bình thường
- ⚠️ "Trường bắt buộc" → đổi label thành **"Bắt buộc tích chọn"** (cùng `isRequired` field name)

### RULE-FIELD-BOOLEAN-03: Mapping form → `meta_data` (`BooleanMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:593-596`.

```typescript
export interface BooleanMetaData {
    false_value?: string;
    true_value?: string;
}
```

| Form key | Đường dẫn trong `meta_data` |
|---|---|
| `trueValue` (textbox "Chọn") | `true_value` |
| `falseValue` (textbox "Không chọn") | `false_value` |

### RULE-FIELD-BOOLEAN-04: `default_value` lưu dạng string

Khác các type khác, `default_value` của boolean lưu trong DB là **string `"true"` / `"false"`** chứ không phải boolean primitive:

```json
{
  "fieldType": "boolean",
  "defaultValue": "true",        // ← string, NOT boolean
  "fieldMetaData": "{\"true_value\":\"Đồng ý\",\"false_value\":\"Từ chối\"}"
}
```

Khi parse ở FE: `Boolean(defaultValue === "true")` hoặc `JSON.parse(defaultValue)`.

### RULE-FIELD-BOOLEAN-05: Payload thực tế (verify từ API response)

Tạo "Test Boolean Field" với default = checked, true_value = "Đồng ý", false_value = "Từ chối":

**Stored field**:

```json
{
  "id": "OFTPOTQE4KM91",
  "slug": "test_boolean_field",
  "name": "Test Boolean Field",
  "fieldType": "boolean",
  "fieldMetaData": "{\"true_value\":\"Đồng ý\",\"false_value\":\"Từ chối\"}",
  "defaultValue": "true",
  "required": 0,
  "multiple": 0,
  "unique": false
}
```

### RULE-FIELD-BOOLEAN-06: Render record

- Khi xem record: hiển thị `true_value` / `false_value` (vd "Đồng ý" / "Từ chối"), không phải "true" / "false".
- Khi edit record: render checkbox 1 ô — checked = true, unchecked = false. Label cạnh checkbox = `name` của field.
- Filter: dùng operator `= true` / `= false`, BE so sánh boolean primitive.

### RULE-FIELD-BOOLEAN-07: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`. Có thể sửa: `name`, `tooltip`, `description`, `isRequired`, `allowManualEdit`, `isActive`, `defaultValue`, `true_value`, `false_value`.
