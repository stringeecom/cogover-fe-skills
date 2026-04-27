---
title: Field Type — Tiền tệ (Currency)
impact: HIGH
impactDescription: Currency field bắt buộc country_code (1 unit duy nhất), khác phone (multi); unit_position đổi vị trí ký hiệu (₫1.000 vs 1.000₫)
tags: data-field, currency, CurrencyMetaData, ui-kit
---

## Field Type — Tiền tệ (Currency)

`fieldType = "currency"` (`DataTypes`). UI label: **"Tiền tệ (Currency)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/currency`.

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-CURRENCY-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Đơn vị tiền tệ khả dụng | `countryCode` | `null` (**bắt buộc nhập**) | Combobox single-select, search 213 đơn vị tiền tệ |
| 2 | Hiển thị đơn vị tiền tệ | `unitPosition` | `"before"` | Radio 2 option: "Trước số tiền" / "Sau số tiền" |
| 3 | Kiểu hiển thị | `displayType` | `Integer` | Combobox (giống Number) |
| 4 | Số lượng ký tự — Phần nguyên | `integralLength` | `15` | NumberInput, default cao hơn Number/Percent (10) |
| 5 | Số lượng ký tự — Phần thập phân | `fractionalLength` | `0` (disabled khi Integer) | |
| 6 | Định dạng hiển thị | `format` | `"Theo cấu hình chung của Workspace"` (`type:1, format:6`) | |
| 7 | Khoảng giá trị — Tối thiểu / Tối đa | `valueMin/Max` | `±999.999.999.999.999` | Auto theo `integralLength` |
| 8 | Giá trị mặc định | `defaultValue` | `""` | |

### RULE-FIELD-CURRENCY-02: Khác biệt với Number/Percent

| Khía cạnh | Number / Percent | Currency |
|---|---|---|
| Default `integral_length` | 10 | **15** |
| Default value range | ±9.999.999.999 | **±999.999.999.999.999** |
| `country_code` | (không có) | **bắt buộc**, single string |
| `unit_position` | (không có) | "before" / "after" |
| `quickSearch` | có (number không, percent ẩn — verify) | (verify) |

### RULE-FIELD-CURRENCY-03: Mapping form → `meta_data` (`CurrencyMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:401-422`.

```typescript
export interface CurrencyMetaData {
    value_limit?: { min?: number; max?: number };
    multiple_limit?: { min?: number; max?: number };
    format?: { type?: number; format?: number };
    round_rule?: NumberRoundRule;
    precision?: number;
    unique_rule?: number;
    integral_length?: number;
    fractional_length?: number;
    display_type?: NumberDisplayType;
    country_code?: string | null;        // ← string, KHÔNG phải array (khác phone)
    unit_position?: "before" | "after" | null;
}
```

| Form key | Đường dẫn trong `meta_data` |
|---|---|
| `countryCode` | `country_code` (lowercase string vd `"vn"`, `"us"`) |
| `unitPosition` | `unit_position` (`"before"` / `"after"`) |
| Còn lại | Giống Number (`display_type`, `integral_length`, `fractional_length`, `value_limit`, `format`, `precision`) |

### RULE-FIELD-CURRENCY-04: Payload thực tế (verify từ API)

Tạo "Test Currency Field" với country_code=VND, unit_position="before", default Integer + integral_length=15:

```json
{
  "fieldType": "currency",
  "fieldMetaData": "{\"value_limit\":{\"min\":-999999999999999,\"max\":999999999999999},\"multiple_limit\":{\"min\":0,\"max\":30},\"format\":{\"type\":1,\"format\":6},\"precision\":-1,\"integral_length\":15,\"fractional_length\":0,\"display_type\":1,\"country_code\":\"vn\",\"unit_position\":\"before\"}"
}
```

**Lưu ý**:
- `country_code` lưu **lowercase** ISO 4217 alpha-3 (`"vn"`, không phải `"VN"`)
- `unit_position`: `"before"` = render `₫1.000`, `"after"` = `1.000₫`
- 213 đơn vị tiền tệ trong combobox (full ISO 4217)

### RULE-FIELD-CURRENCY-05: Single-select country (khác Phone)

Khác `phone` (multi-select `country_code: string[]`), `currency` chỉ chọn **1 đơn vị tiền tệ duy nhất** cho cả field. User không thể nhập USD và VND lẫn lộn — phải tạo 2 field riêng nếu muốn lưu nhiều đơn vị.

### RULE-FIELD-CURRENCY-06: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`. Có thể sửa: `country_code`, `unit_position`, `integral_length`, `fractional_length`, `display_type`, `format`, `value_limit`, `defaultValue`. Đổi `country_code` không đổi giá trị số đã lưu — chỉ đổi cách hiển thị.
