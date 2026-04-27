---
title: Field Type — Văn bản ngắn (Short-text)
impact: HIGH
impactDescription: Sai cấu hình character_limit hoặc display_type khiến input render lệch (textarea thay 1-line) hoặc validation lỗi
tags: data-field, short-text, short_text, ShortTextMetaData, ui-kit
---

## Field Type — Văn bản ngắn (Short-text)

`fieldType = "short_text"` (`DataTypes`). UI label: **"Văn bản ngắn (Short-text)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/short-text`.

Cấu hình common: xem `field-types/common`. File này chỉ liệt kê **phần riêng** của short-text.

### RULE-FIELD-SHORTTEXT-01: Form fields riêng

Bổ sung **3 nhóm** input ngoài common (RULE-FIELD-COMMON-02):

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Số lượng ký tự cho phép nhập — Tối thiểu | `minLength` | `0` | NumberInput, ≥ 0 |
| 2 | Số lượng ký tự cho phép nhập — Tối đa | `maxLength` | `255` | NumberInput, ≥ `minLength` |
| 3 | Kiểu hiển thị | `displayType` | `"Văn bản ngắn (1 dòng)"` | Combobox 2 option (xem RULE-02) |

Khi `isMultipleValues = true` (RULE-FIELD-COMMON-04), thêm 2 input nhập `multiple_limit.min` / `multiple_limit.max`.

### RULE-FIELD-SHORTTEXT-02: Combobox `displayType` chỉ có 2 option

| UI label | Giá trị enum (`ShortTextDisplayTypes`) | Render trong record |
|---|---|---|
| Văn bản ngắn (1 dòng) | `OneLine = 1` | `<input type="text">` (1 dòng) |
| Đoạn văn bản (nhiều dòng) | `MultipleLines = 2` | `<textarea>` (nhiều dòng) |

> **Lưu ý**: dù chọn `MultipleLines`, fieldType **vẫn là** `short_text` (không phải `long_text`). Khác biệt với `long_text` là không có rich text / markdown.

Tooltip giải thích dưới combobox: `"Văn bản ngắn (1 dòng) hỗ trợ tìm kiếm tương đối (LIKE)."`

### RULE-FIELD-SHORTTEXT-03: Mapping form → `meta_data` (`ShortTextMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:224-229`.

```typescript
export interface ShortTextMetaData {
    character_limit?: { min?: number; max?: number };
    multiple_limit?: { min?: number; max?: number };
    display_type: ShortTextDisplayTypes; // 1 = OneLine, 2 = MultipleLines
    unique_rule?: number;
}

export enum ShortTextDisplayTypes {
    OneLine = 1,
    MultipleLines = 2,
}
```

| Form key | Đường dẫn trong `meta_data` |
|---|---|
| `minLength` | `character_limit.min` |
| `maxLength` | `character_limit.max` |
| `displayType` ("Văn bản ngắn (1 dòng)" / "Đoạn văn bản (nhiều dòng)") | `display_type` (1 / 2) |
| `multiple_limit.min` (nếu `isMultipleValues`) | `multiple_limit.min` |
| `multiple_limit.max` (nếu `isMultipleValues`) | `multiple_limit.max` |
| (không có UI) | `unique_rule` — không set, BE chỉ dùng top-level `unique` |

### RULE-FIELD-SHORTTEXT-04: Payload thực tế (verify từ network)

Submit form với: `name="Test Short Text Field"`, `isRequired=true`, `quickSearch=true`, `displayType=OneLine`, `defaultValue="default text"`, đã capture được:

```json
POST /api/v1/objects/field/create
{
  "type": "short_text",
  "object_type_id": "OTTPOTQETKMSA",
  "workspace_id": "WSsgfBnZoNrPA",
  "name": "Test Short Text Field",
  "slug": "test_short_text_field",
  "tool_tip": "Tooltip mô tả ngắn cho field",
  "hint_text": "Nhập text test",
  "description": "Mô tả field test ngắn",
  "default_value": "default text",
  "required": 1,
  "manual_modify_allow": true,
  "status": 1,
  "quickSearch": 1,
  "multiple": 0,
  "unique": 0,
  "meta_data": "{\"character_limit\":{\"min\":1,\"max\":255},\"multiple_limit\":{\"min\":1,\"max\":30},\"display_type\":1}"
}
```

Response (success):

```json
{
  "tags": [],
  "r": 0,
  "msg": "OK",
  "requestId": "RQ20dfe3ba-3849-4c7e-ac05-5ffce5fd5320",
  "data": [{ "id": "OFTPOTQEDKM9Y", "slug": "test_short_text_field", "options": [] }],
  "meta": null
}
```

### RULE-FIELD-SHORTTEXT-05: Hành vi tự động khi `isRequired = true`

Khi check "Trường bắt buộc":
- FE tự **set `character_limit.min = 1`** (từ default 0) — đảm bảo bắt buộc nhập ít nhất 1 ký tự.
- Khi user uncheck → min vẫn giữ giá trị 1 (không reset về 0), user phải tự sửa lại nếu muốn.

### RULE-FIELD-SHORTTEXT-06: Default `character_limit`

- FE preset ban đầu: `min = 0`, `max = 255` (khớp giới hạn DB của short_text).
- Sau khi check `isRequired`: `min = 1`.
- User có thể giảm `max` xuống nhưng **không tăng quá 255** — UI sẽ disable nút tăng (đã thấy 2 button `disabled` cạnh `maxLength` trong DOM).

### RULE-FIELD-SHORTTEXT-07: Default `multiple_limit`

Khi `multiple = 0` (single value), payload **vẫn** có `multiple_limit: { min: 1, max: 30 }` (chú ý max=30 — giá trị FE preset, không phải 1). BE bỏ qua block này khi `multiple = 0`. Xem RULE-FIELD-COMMON-04.

### RULE-FIELD-SHORTTEXT-08: Lưu ý khi sửa

Sau khi tạo, các thuộc tính **không sửa được**: `slug`, `type`. Có thể sửa: `name`, `tooltip`, `hint`, `description`, `isRequired`, `allowManualEdit`, `isActive`, `quickSearch`, `isMultipleValues`, `isUnique`, `character_limit`, `display_type`, `defaultValue`.
