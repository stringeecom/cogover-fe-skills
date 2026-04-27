---
title: Field Type — Số (Number / Numeric / Decimal)
impact: HIGH
impactDescription: Sai display_type / integral_length / fractional_length gây overflow hoặc validation sai khi nhập số thập phân
tags: data-field, number, numeric, decimal, NumberMetaData, ui-kit
---

## Field Type — Số (Number)

`fieldType = "numeric"` (`DataTypes`) — UI label: **"Số (Number)"**.

⚠️ Trên UI chỉ có **1 entry** "Số (Number)" với combobox `display_type` chuyển integer ↔ decimal. Field type lưu DB **luôn là** `"numeric"`. `"decimal"` trong `DataTypes` enum là **legacy / unused** từ form tạo mới.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/number`.

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-NUMBER-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Kiểu hiển thị | `displayType` | `"Số nguyên (Integer)"` | Combobox 2 option (xem RULE-02) |
| 2 | Số lượng ký tự tối đa — Phần nguyên | `integralLength` | `10` | NumberInput, số chữ số phần nguyên |
| 3 | Số lượng ký tự tối đa — Phần thập phân | `fractionalLength` | `0` (disabled khi Integer) | NumberInput, số chữ số phần thập phân |
| 4 | Định dạng hiển thị | `format` | `"Theo cấu hình chung của Workspace"` (`type:1, format:6`) | Combobox — chọn separator/decimal char |
| 5 | Khoảng giá trị — Tối thiểu | `valueMin` | `-9.999.999.999` (auto theo `integralLength`) | Min value |
| 6 | Khoảng giá trị — Tối đa | `valueMax` | `9.999.999.999` (auto theo `integralLength`) | Max value |
| 7 | Giá trị mặc định | `defaultValue` | `""` | Số mặc định |
| 8 | Quy tắc làm tròn | `roundRule` | (ẩn khi Integer) | Combobox khi Decimal: None / Round / Up / Down |

### RULE-FIELD-NUMBER-02: Combobox `displayType` — `NumberDisplayType` enum

| UI label | Enum value |
|---|---|
| Số nguyên (Integer) | `Integer = 1` |
| Số thập phân (Decimal) | `Decimal = 2` |

### RULE-FIELD-NUMBER-03: Input common bị ẩn

So với common (RULE-FIELD-COMMON-02):
- ❌ KHÔNG có "Cho phép sử dụng để Tìm kiếm nhanh" (`quickSearch`) — số không hỗ trợ quick search

### RULE-FIELD-NUMBER-04: Mapping form → `meta_data` (`NumberMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:320-338`.

```typescript
export interface NumberMetaData {
    value_limit?: { min?: number; max?: number };
    multiple_limit?: { min?: number; max?: number };
    format?: { type?: number; format?: number };
    round_rule?: NumberRoundRule;
    precision?: number;
    unique_rule?: number;
    integral_length?: number;
    fractional_length?: number;
    // Bổ sung khi sync (verify từ payload thực tế):
    // display_type?: NumberDisplayType;
}

export enum NumberDisplayType { Integer = 1, Decimal = 2 }
export enum NumberRoundRule { None = 0, Round = 1, RoundUp = 2, RoundDown = 3 }
```

| Form key | Đường dẫn trong `meta_data` |
|---|---|
| `displayType` | `display_type` ⚠️ key này **chưa có trong `NumberMetaData`** (kế thừa từ `PercentMetaData`) |
| `integralLength` | `integral_length` |
| `fractionalLength` | `fractional_length` |
| `format` | `format: { type, format }` |
| `valueMin` / `valueMax` | `value_limit.min` / `value_limit.max` |
| `roundRule` | `round_rule` |
| (tự tính từ `fractionalLength`) | `precision` (`-1` khi Integer / hoặc `fractionalLength` khi Decimal) |

### RULE-FIELD-NUMBER-05: Payload thực tế (verify từ API)

Tạo "Test Number Field" với default Integer, integral_length=10, không sửa gì:

```json
{
  "fieldType": "numeric",
  "fieldMetaData": "{\"value_limit\":{\"min\":-9999999999,\"max\":9999999999},\"multiple_limit\":{\"min\":0,\"max\":30},\"format\":{\"type\":1,\"format\":6},\"precision\":-1,\"integral_length\":10,\"fractional_length\":0,\"display_type\":1}",
  "defaultValue": "",
  "multiple": 0,
  "unique": false
}
```

Quan sát:
- `value_limit.min/max` **auto-set** theo `integral_length=10` → `±9999999999` (10 chữ số 9).
- `display_type: 1` (Integer mode)
- `precision: -1` khi Integer (không làm tròn)
- `format: { type: 1, format: 6 }` = "Theo cấu hình chung của Workspace" (default)

### RULE-FIELD-NUMBER-06: Khi switch sang Decimal mode

- `display_type` = `2`
- `fractional_length` enabled, default 2 (FE preset, có thể tăng)
- `value_limit` mở rộng: `±9999999999.99` (theo integral + fractional)
- `precision` = `fractional_length`
- Hiện thêm UI "Quy tắc làm tròn" (`round_rule`)

### RULE-FIELD-NUMBER-07: `format` combobox

`format: { type, format }` — encoding kiểu hiển thị thousand-separator + decimal-char:
- `type: 0` = Custom (user chọn riêng)
- `type: 1` = "Theo cấu hình chung của Workspace" (default)
- `format: 6` = Đại diện cho format mặc định của workspace VN (vd `1.234.567,89`)

Các giá trị `format` khác (0-5) tương ứng các locale: US (`1,234,567.89`), EU (`1 234 567,89`)... Cần verify thêm khi đổi UI.

### RULE-FIELD-NUMBER-08: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`. Có thể sửa: `display_type`, `integral_length`, `fractional_length`, `value_limit`, `format`, `round_rule`, `defaultValue`. Chuyển Integer → Decimal **không** mất dữ liệu (số nguyên vẫn lưu được trong cột decimal); ngược lại có thể truncate.
