---
title: Field Type — Đường dẫn liên kết URL
impact: MEDIUM
impactDescription: use_display_text đổi cách render URL: text alias vs raw URL; sai gây UX khó click link
tags: data-field, url, UrlMetaData, ui-kit
---

## Field Type — Đường dẫn liên kết URL

`fieldType = "url"` (`DataTypes`). UI label: **"Đường dẫn liên kết URL"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/url`.

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-URL-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Số lượng ký tự — Tối thiểu | `minLength` | `0` | NumberInput |
| 2 | Số lượng ký tự — Tối đa | `maxLength` | `2048` | NumberInput, max 2048 (chuẩn URL) |
| 3 | Sử dụng văn bản thay thế | `useDisplayText` | `false` | Checkbox — bật cho phép user nhập **alias text** thay vì hiển thị URL thô |

Khi `isMultipleValues = true`: hiện thêm 2 input `multiple_limit.min` / `multiple_limit.max`.

### RULE-FIELD-URL-02: Input common bị ẩn

So với common (RULE-FIELD-COMMON-02):
- ❌ KHÔNG có "Giá trị mặc định" (`defaultValue`) — URL không hỗ trợ default value (do alias text 2 thành phần phức tạp)

### RULE-FIELD-URL-03: Mapping form → `meta_data` (`UrlMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:274-279`.

```typescript
export interface UrlMetaData {
    character_limit?: { min?: number; max?: number };
    multiple_limit?: { min?: number; max?: number };
    use_display_text?: boolean;
    unique_rule?: number;
}
```

| Form key | Đường dẫn trong `meta_data` |
|---|---|
| `minLength` | `character_limit.min` |
| `maxLength` | `character_limit.max` |
| `useDisplayText` | `use_display_text` |
| `multiple_limit.min/max` (khi `isMultipleValues`) | `multiple_limit.min/max` |

### RULE-FIELD-URL-04: Payload thực tế (verify từ API)

Tạo "Test URL Field", bật "Sử dụng văn bản thay thế":

```json
{
  "id": "OF0UCCQE6KMSJ",
  "slug": "test_url_field",
  "fieldType": "url",
  "fieldMetaData": "{\"character_limit\":{\"min\":0,\"max\":2048},\"multiple_limit\":{\"min\":0,\"max\":30},\"use_display_text\":true}",
  "required": 0,
  "multiple": 0,
  "unique": false,
  "defaultValue": ""
}
```

### RULE-FIELD-URL-05: Value structure ở Record (`UrlData`)

Khi `use_display_text = true`, value của record lưu dạng:

```typescript
export interface UrlData {
    url?: string;     // URL thực tế (https://...)
    alias?: string;   // Văn bản thay thế hiển thị (vd "Trang chủ Stringee")
}
```

Khi `use_display_text = false`, value chỉ lưu URL string.

### RULE-FIELD-URL-06: Render record

- `use_display_text = true`: hiển thị `<a href={url}>{alias || url}</a>`.
- `use_display_text = false`: hiển thị `<a href={url}>{url}</a>`.
- Cả 2 trường hợp đều mở tab mới (`target="_blank"`).

### RULE-FIELD-URL-07: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`. Có thể sửa: `use_display_text`, `character_limit`, `multiple_limit`. Chuyển từ `use_display_text=false` → `true` không tự backfill alias cho record cũ — cần migrate riêng.
