---
title: Field Type — Đánh số tự động (Auto-number)
impact: HIGH
impactDescription: is_identifier biến field thành slug bản ghi; is_auto_fill_missed_data backfill bản ghi cũ; format quy định padding zero
tags: data-field, auto-number, auto_number, AutoNumberMetaData, ui-kit
---

## Field Type — Đánh số tự động (Auto-number)

`fieldType = "auto_number"` (`DataTypes`). UI label: **"Đánh số tự động (Auto-number)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/auto-number`.

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-AUTONUMBER-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Sử dụng trường để định danh URL cho bản ghi | `isIdentifier` | `false` | Checkbox — bật field thay thế `id` trong URL bản ghi |
| 2 | Tiền tố | `prefix` | `""` | Textbox — chuỗi đứng trước số |
| 3 | Hậu tố | `suffix` | `""` | Textbox — chuỗi đứng sau số |
| 4 | Định dạng số tự động | `format` | `"0000"` | Textbox **bắt buộc** — pattern padding zero (`"0000"` → 4 chữ số: 0001, 0042) |
| 5 | Số bắt đầu | `start` | `1` | NumberInput |
| 6 | Bước nhảy | `step` | `1` | NumberInput |
| 7 | Tạo số tự động cho các bản ghi đã có sẵn | `isAutoFillMissedData` | `false` | Checkbox — backfill record cũ chưa có số (one-time apply) |

### RULE-FIELD-AUTONUMBER-02: Input common bị ẩn

So với common (RULE-FIELD-COMMON-02):
- ❌ KHÔNG có Tooltip (`tooltip`)
- ❌ KHÔNG có Placeholder (`hint`)
- ❌ KHÔNG có "Trường bắt buộc" (`isRequired`) — auto-gen luôn có giá trị
- ❌ KHÔNG có "Cho phép tạo, sửa thủ công" (`allowManualEdit`) — system-managed
- ❌ KHÔNG có "Cho phép trường có nhiều giá trị" (`isMultipleValues`)
- ❌ KHÔNG có "Tính duy nhất" (`isUnique`) — auto-number always unique
- ❌ KHÔNG có "Giá trị mặc định" (`defaultValue`)

### RULE-FIELD-AUTONUMBER-03: Mapping form → `meta_data` (`AutoNumberMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:364-371`.

```typescript
export interface AutoNumberMetaData {
    prefix?: string;
    suffix?: string;
    format?: string;
    start?: number;
    step?: number;
    is_identifier?: boolean;
    // Bổ sung khi sync (verify từ payload):
    // is_auto_fill_missed_data?: boolean;
}
```

| Form key | Đường dẫn trong `meta_data` |
|---|---|
| `prefix` | `prefix` |
| `suffix` | `suffix` |
| `format` | `format` |
| `start` | `start` |
| `step` | `step` |
| `isIdentifier` | `is_identifier` |
| `isAutoFillMissedData` | `is_auto_fill_missed_data` ⚠️ key này **chưa có trong `AutoNumberMetaData`** |

### RULE-FIELD-AUTONUMBER-04: Payload thực tế (verify từ API)

Tạo "Test Auto Number Field" với prefix="CUS-", default config khác:

```json
{
  "fieldType": "auto_number",
  "fieldMetaData": "{\"prefix\":\"CUS-\",\"suffix\":\"\",\"format\":\"0000\",\"start\":1,\"step\":1,\"is_identifier\":false,\"is_auto_fill_missed_data\":false}"
}
```

Sequence sinh: `CUS-0001`, `CUS-0002`, `CUS-0003`...

### RULE-FIELD-AUTONUMBER-05: `is_identifier` — định danh URL

- `false` (default): URL bản ghi dùng `id` system (vd `/object/customer/RECxxx`)
- `true`: URL dùng giá trị field này (vd `/object/customer/CUS-0001`)

Chỉ 1 field auto-number trong object được phép `is_identifier = true`. Nếu đã có field khác đang là identifier → bật field mới sẽ tự tắt field cũ (verify behavior — chưa test 2 field).

### RULE-FIELD-AUTONUMBER-06: Liên kết với `name_field` của object

Khi tạo Object với `name_field.type = auto_number` (xem `data-model/object` RULE-OBJECT-09), object tự sinh field `name` với metadata = `name_field.meta_data`. Cấu hình của field `name` đó **giống** form auto-number ở đây nhưng read-only sau khi tạo (xem RULE-OBJECT-11).

### RULE-FIELD-AUTONUMBER-07: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`, `format` (ảnh hưởng đến giá trị đã sinh). Có thể sửa: `prefix`, `suffix`, `step`, `is_identifier`. Sửa `prefix`/`suffix` chỉ ảnh hưởng record mới; record cũ giữ giá trị cũ trừ khi bật `is_auto_fill_missed_data` (verify behavior).
