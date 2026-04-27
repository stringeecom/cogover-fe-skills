---
title: Field Type — Email
impact: MEDIUM
impactDescription: Email field auto-validate format ở FE; sai metadata gây lỗi nhập email không hợp lệ
tags: data-field, email, EmailMetaData, ui-kit
---

## Field Type — Email

`fieldType = "email"` (`DataTypes`). UI label: **"Email"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/email`.

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-EMAIL-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Số lượng ký tự — Tối thiểu | `minLength` | `0` | NumberInput |
| 2 | Số lượng ký tự — Tối đa | `maxLength` | `255` | NumberInput, max 255 |
| 3 | Giá trị mặc định | `defaultValue` | `""` | Textbox, placeholder mẫu `"johndoe@gmail.com"` |

Khi `isMultipleValues = true`: hiện thêm 2 input `multiple_limit.min` / `multiple_limit.max`.

### RULE-FIELD-EMAIL-02: Input common đầy đủ

Email có **đủ 12 input common** trong RULE-FIELD-COMMON-02 (không ẩn / khoá gì).

### RULE-FIELD-EMAIL-03: Mapping form → `meta_data` (`EmailMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:281-285`.

```typescript
export interface EmailMetaData {
    character_limit?: { min?: number; max?: number };
    multiple_limit?: { min?: number; max?: number };
    unique_rule?: number;
}
```

| Form key | Đường dẫn trong `meta_data` |
|---|---|
| `minLength` | `character_limit.min` |
| `maxLength` | `character_limit.max` |
| `multiple_limit.min` (khi `isMultipleValues`) | `multiple_limit.min` |
| `multiple_limit.max` (khi `isMultipleValues`) | `multiple_limit.max` |

### RULE-FIELD-EMAIL-04: Payload thực tế (verify từ API response)

Tạo "Test Email Field" với chỉ name, không cấu hình thêm:

```json
{
  "id": "OF0UCCQEEKMSB",
  "slug": "test_email_field",
  "fieldType": "email",
  "fieldMetaData": "{\"character_limit\":{\"min\":0,\"max\":255},\"multiple_limit\":{\"min\":0,\"max\":30}}",
  "required": 0,
  "multiple": 0,
  "unique": false,
  "defaultValue": ""
}
```

⚠️ **Khác với short_text**: `multiple_limit.min` mặc định là **`0`** cho email (cho phép record không có email), trong khi short_text là `1`.

### RULE-FIELD-EMAIL-05: Validate format

FE và BE đều validate format email (regex tiêu chuẩn `name@domain.tld`). Khi BE từ chối, response trả lỗi format. FE check trước khi submit để tránh round-trip không cần thiết.

### RULE-FIELD-EMAIL-06: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`. Có thể sửa: tất cả còn lại bao gồm `character_limit`, `multiple_limit`, `defaultValue`.
