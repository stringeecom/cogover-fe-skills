---
title: Field Type — Biểu thức chính quy (Regex)
impact: HIGH
impactDescription: regex pattern bắt buộc; type def dùng value_limit nhưng API thực tế lưu character_limit (sync mismatch)
tags: data-field, regex, RegexMetaData, ui-kit
---

## Field Type — Biểu thức chính quy (Regex)

`fieldType = "regex"` (`DataTypes`). UI label: **"Biểu thức chính quy (Regex)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/regex`.

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-REGEX-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Biểu thức chính quy | `regex` | `""` | Textbox **bắt buộc** — pattern regex |
| 2 | Kiểm tra biểu thức | `regexTest` | `""` | Textbox preview (không lưu vào meta_data) |
| 3 | Giá trị mặc định | `defaultValue` | `""` | Textbox |

### RULE-FIELD-REGEX-02: Mapping form → `meta_data` (`RegexMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:424-435`.

```typescript
export interface RegexMetaData {
    value_limit?: { min?: number; max?: number };  // ⚠️ KHÔNG khớp với API
    multiple_limit?: { min?: number; max?: number };
    regex?: string;
    unique_rule?: number;
}
```

⚠️ **Type def vs Reality mismatch**: Type def có `value_limit` nhưng BE thực tế lưu `character_limit`. Nên cập nhật type khi sync.

| Form key | Đường dẫn trong `meta_data` (thực tế) |
|---|---|
| `regex` | `regex` |
| (auto-set?) | `character_limit.min/max` (default 0/255) |
| (UI ẩn) | `multiple_limit.min/max` |

### RULE-FIELD-REGEX-03: Payload thực tế (verify từ API)

Tạo "Test Regex Field" với pattern `^[A-Z]{2}-\d{4}$`:

```json
{
  "fieldType": "regex",
  "fieldMetaData": "{\"character_limit\":{\"min\":0,\"max\":255},\"multiple_limit\":{\"min\":0,\"max\":30},\"regex\":\"^[A-Z]{2}-\\\\d{4}$\"}"
}
```

Pattern bị escape JSON 2 lần: `\d` → `\\d` trong JSON string.

### RULE-FIELD-REGEX-04: Render record

User nhập input thường, FE/BE validate qua `RegExp(meta_data.regex)`. Lỗi format → block submit. KHÔNG render input mask đặc biệt.

### RULE-FIELD-REGEX-05: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`. Có thể sửa: `regex`, `character_limit`, `multiple_limit`. Đổi `regex` không re-validate record cũ — record sai format sẽ vẫn lưu cho đến khi user edit.
