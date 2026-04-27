---
title: Field Type — Số điện thoại (Phone)
impact: HIGH
impactDescription: country_code limit số được phép, is_show_button_call thay đổi UI list (nút gọi); duplicate_check_type chưa có trong PhoneNumberMetaData type
tags: data-field, phone, PhoneNumberMetaData, ui-kit
---

## Field Type — Số điện thoại (Phone)

`fieldType = "phone"` (`DataTypes`). UI label: **"Số điện thoại (Phone)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/phone-number` ⚠️ (slug `phone-number`, không phải `phone`).

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-PHONE-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Hiển thị nút gọi trên màn danh sách | `isShowButtonCall` | `false` | Checkbox — bật cho phép user click "gọi điện" trực tiếp ở list view |
| 2 | Kiểm tra là số điện thoại hợp lệ | `isValidateFormat` | `false` | Checkbox — bật bắt buộc phone hợp lệ theo pattern `country_code` |
| 3 | Mã vùng khả dụng | `countryCode` | **All 208 countries** | Multi-select — danh sách quốc gia user được phép chọn khi nhập phone |
| 4 | Kiểu kiểm tra trùng lặp | `duplicateCheckType` | `1` ("Tuyệt đối") | Combobox — quy tắc check unique |

### RULE-FIELD-PHONE-02: Input common bị ẩn

So với common (RULE-FIELD-COMMON-02):
- ❌ KHÔNG có "Số lượng ký tự" (`character_limit`) — phone validate qua `country_code`/`regex`, không qua length
- ❌ KHÔNG có "Giá trị mặc định" (`defaultValue`)

### RULE-FIELD-PHONE-03: Mapping form → `meta_data` (`PhoneNumberMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:251-263`.

```typescript
export interface PhoneNumberMetaData {
    multiple_limit?: { min?: number; max?: number };
    country_code?: string[];          // ISO alpha-2 codes (vd ["VN", "US", "JP"])
    unique_rule?: number;
    regex?: string;
    is_show_button_call?: number;     // 0 | 1 — KHÔNG phải boolean
    is_show_button_call_layout?: boolean;
    is_show_when_hover?: boolean;
    is_validate_format?: boolean;
    // Chưa có trong type def — bổ sung khi sync:
    // duplicate_check_type?: number;
}
```

| Form key | Đường dẫn trong `meta_data` |
|---|---|
| `isShowButtonCall` | `is_show_button_call` (`number` 0/1) |
| `isValidateFormat` | `is_validate_format` (`boolean`) |
| `countryCode` (multi-select) | `country_code: string[]` |
| `duplicateCheckType` | `duplicate_check_type` ⚠️ key này **chưa có trong `PhoneNumberMetaData`** |
| `multiple_limit.min/max` | `multiple_limit.min/max` |
| (UI ẩn) | `regex` (rỗng mặc định) |

### RULE-FIELD-PHONE-04: 208 country codes mặc định

Bật tất cả 208 quốc gia khi tạo mới. Danh sách rút gọn: `["AF", "AL", "DZ", "AD", "AO", ..., "VN", ..., "ZW"]` (ISO 3166-1 alpha-2).

User có thể bỏ chọn 1 số nước → chỉ giữ những nước được phép. UI hiển thị "+208" badge khi chọn full.

### RULE-FIELD-PHONE-05: `duplicate_check_type` — combobox "Kiểu kiểm tra trùng lặp"

Combobox 2 option (verify từ UI):
- `1` = **Tuyệt đối** (Absolute) — match toàn bộ chuỗi (cùng `+84 988 999 999`)
- `2` = **Tương đối** (Relative) — match digits, bỏ qua format/space (`+84988999999` = `0988-999-999`)

Default = `1`. Bắt buộc nhập (UI có dấu `*`).

### RULE-FIELD-PHONE-06: Payload thực tế (verify từ API)

Tạo "Test Phone Field" với is_show_button_call + is_validate_format đều bật, country_code default (208), duplicate_check_type default (1):

```json
{
  "fieldType": "phone",
  "fieldMetaData": "{\"country_code\":[\"AF\",\"AL\",...208 codes...,\"ZW\"],\"multiple_limit\":{\"min\":0,\"max\":30},\"regex\":\"\",\"is_show_button_call\":1,\"is_validate_format\":true,\"duplicate_check_type\":1}",
  "defaultValue": "",
  "multiple": 0,
  "unique": false
}
```

⚠️ **Khác biệt giữa boolean & number trong meta_data**:
- `is_show_button_call`: **`1`** (number)
- `is_validate_format`: **`true`** (boolean)

→ Khi parse meta_data ở FE, cần dùng `Boolean(value)` hoặc `value === 1` tuỳ key. Không assume cùng kiểu.

### RULE-FIELD-PHONE-07: Render record

- Khi nhập: render `<PhoneInput>` từ `@stringeecom/ui-kit` với combobox country_code prefix bị giới hạn theo `country_code` array.
- Khi xem (list view): nếu `is_show_button_call = 1` → hiển thị nút "📞 Gọi" cạnh số.
- `is_validate_format = true` → block submit nếu không hợp lệ.

### RULE-FIELD-PHONE-08: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`. Có thể sửa: `country_code`, `is_show_button_call`, `is_validate_format`, `duplicate_check_type`, `multiple_limit`. Đổi `country_code` không backfill validate cho record cũ → cần migrate.
