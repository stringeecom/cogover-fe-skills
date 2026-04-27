---
title: Field Type — Phần trăm (Percent)
impact: MEDIUM
impactDescription: Form gần như giống Number; metadata structure trùng nhưng field "%" hiển thị trong UI render khác
tags: data-field, percent, PercentMetaData, ui-kit
---

## Field Type — Phần trăm (Percent)

`fieldType = "percent"` (`DataTypes`). UI label: **"Phần trăm (Percent)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/percent`.

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-PERCENT-01: Form fields giống `number`

Form **giống hoàn toàn** Number (xem `number.md` RULE-FIELD-NUMBER-01) bao gồm:
- `displayType` (Integer/Decimal)
- `integralLength` / `fractionalLength`
- `format` combobox
- `valueMin` / `valueMax`
- `defaultValue`
- `roundRule` (khi Decimal)

Khác duy nhất: render record có hậu tố **`%`**, value lưu DB là số thực (vd `0.15` = 15%, hoặc `15` tuỳ implementation BE).

### RULE-FIELD-PERCENT-02: Mapping form → `meta_data` (`PercentMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:299-318`.

```typescript
export interface PercentMetaData {
    value_limit?: { min?: number; max?: number };
    multiple_limit?: { min?: number; max?: number };
    format: { type: number; format: number };
    round_rule: NumberRoundRule;
    precision: number;
    unique_rule?: number;
    integral_length: number;
    fractional_length: number;
    display_type: NumberDisplayType;
}
```

Khác `NumberMetaData`: nhiều field **không optional** (`format`, `round_rule`, `precision`, `integral_length`, `fractional_length`, `display_type` đều required) — nghĩa là FE **bắt buộc** gửi đủ.

Schema mapping giống số (RULE-FIELD-NUMBER-04).

### RULE-FIELD-PERCENT-03: Payload thực tế (verify từ API)

Tạo "Test Percent Field" với default Integer + integral_length=10:

```json
{
  "fieldType": "percent",
  "fieldMetaData": "{\"value_limit\":{\"min\":-9999999999,\"max\":9999999999},\"multiple_limit\":{\"min\":0,\"max\":30},\"format\":{\"type\":1,\"format\":6},\"precision\":-1,\"integral_length\":10,\"fractional_length\":0,\"display_type\":1}"
}
```

Trùng schema với `numeric` — chỉ khác `fieldType: "percent"`.

### RULE-FIELD-PERCENT-04: Khác biệt với `numeric`

| Khía cạnh | `numeric` | `percent` |
|---|---|---|
| `fieldType` | `"numeric"` | `"percent"` |
| Render record | Số thuần | Số kèm `%` |
| Value lưu | Số thực | Số thực (BE quy định ý nghĩa: 0.15 hay 15) |
| Combobox tạo | "Số (Number)" | "Phần trăm (Percent)" |
| Use case | Số lượng, tuổi, điểm | Tỷ lệ, hoa hồng, discount |

### RULE-FIELD-PERCENT-05: Lưu ý khi sửa

Tương tự `numeric` (RULE-FIELD-NUMBER-08).
