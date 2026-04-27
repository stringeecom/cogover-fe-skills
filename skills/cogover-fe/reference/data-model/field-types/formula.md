---
title: Field Type — Công thức (Formula)
impact: HIGH
impactDescription: return_type quyết định kiểu giá trị (text/number/boolean/date); calculation_mode chuyển on-view vs on-write; field luôn read-only ở record
tags: data-field, formula, FormulaMetaData, ui-kit
---

## Field Type — Công thức (Formula)

`fieldType = "formula"` (`DataTypes`). UI label: **"Công thức (Formula)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/formula`.

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-FORMULA-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Chạy Formula lúc xem bản ghi / Chạy khi tạo hoặc sửa | `calculationMode` | `0` (xem) | Radio 2 option (xem RULE-04) |
| 2 | Kiểu trả về của công thức | `returnType` | `"Chữ"` (`Text = 1`) | Combobox 7 option theo `FormulaReturnType` |
| 3 | Thiết lập công thức | `script` | `""` (bắt buộc) | Code editor với syntax highlighting + functions panel |
| 4 | Hiển thị giá trị trả về dưới dạng HTML | `displayHtml` | `false` | Checkbox (chỉ khi `returnType = Text`) |
| 5 | Xử lý trường trống — Xem như giá trị 0 | `zeroDefaultValue` | `true` | Checkbox |

### RULE-FIELD-FORMULA-02: Input common bị ẩn / khoá

So với common (RULE-FIELD-COMMON-02):
- ❌ KHÔNG có Placeholder (`hint`)
- ❌ KHÔNG có "Trường bắt buộc" (`isRequired`) — formula auto-compute, không nhập tay
- ❌ KHÔNG có "Cho phép tạo, sửa thủ công" (`allowManualEdit`) — luôn read-only
- ⚠️ "Cho phép sử dụng để Tìm kiếm nhanh" (`quickSearch`) → **disabled** (formula không index search)
- ❌ KHÔNG có "Cho phép trường có nhiều giá trị" (`isMultipleValues`)
- ❌ KHÔNG có "Tính duy nhất" (`isUnique`)
- ❌ KHÔNG có "Giá trị mặc định" (`defaultValue`)

### RULE-FIELD-FORMULA-03: Combobox `returnType` — `FormulaReturnType` enum

```typescript
export enum FormulaReturnType {
    Text = 1,
    Number = 2,
    Boolean = 8,        // ⚠️ giá trị 8, KHÔNG phải 3
    DateTime = 4,
    Date = 5,
    DateTimeRange = 6,
    DateRange = 7,
}
```

Khi `returnType = Number`:
- Hiện thêm config `number_type` (Normal/Currency/Percent), `format`, `fractional_length`, `currency_code`, `unit_position`.

Khi `returnType = Boolean`:
- Hiện thêm `display_type` (Checkbox/AlternativeText), `alternative_text_true/false`.

### RULE-FIELD-FORMULA-04: Combobox `calculationMode`

```typescript
calculation_mode: 0 | 1
```

| Value | Behavior |
|---|---|
| `0` | **On-view**: tính lại mỗi lần load record (real-time, dynamic) |
| `1` | **On-write**: tính khi tạo/sửa record liên quan, lưu cache (static, faster read) |

### RULE-FIELD-FORMULA-05: Mapping form → `meta_data` (`FormulaMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:654-670`.

```typescript
export interface FormulaMetaData {
    alternative_text_false?: string;
    alternative_text_true?: string;
    display_type?: FormulaBooleanDisplayTypes;  // chỉ khi return_type = Boolean
    return_type: FormulaReturnType;
    script: string;
    format: string;
    fractional_length: number;
    number_type?: FormulaNumberType;            // chỉ khi return_type = Number
    currency_code: string;
    unit_position: "before" | "after" | null;
    range_date: boolean;
    display_html: boolean;
    zero_default_value: boolean;
    time_zone?: string;                         // ⚠️ verify từ payload
    calculation_mode: 0 | 1;
}

export enum FormulaBooleanDisplayTypes { Checkbox = 1, AlternativeText = 2 }
export enum FormulaNumberType { Normal = 1, Currency = 2, Percent = 3 }
```

| Form key | Đường dẫn |
|---|---|
| `script` | `meta_data.script` |
| `returnType` | `meta_data.return_type` (1/2/4/5/6/7/8) |
| `calculationMode` | `meta_data.calculation_mode` (0/1) |
| `displayHtml` | `meta_data.display_html` |
| `zeroDefaultValue` | `meta_data.zero_default_value` |
| (auto-set) | `meta_data.time_zone` (vd `"Asia/Manila"` lấy từ workspace) |
| (auto-set) | `meta_data.range_date` (false default) |
| (auto-set) | `meta_data.meta` (empty object — bổ sung) |

### RULE-FIELD-FORMULA-06: Payload thực tế (verify từ API)

Tạo "Test Formula Field" với script `"Hello World"`, returnType=Text default, calculationMode=on-view:

```json
{
  "fieldType": "formula",
  "fieldMetaData": "{\"return_type\":1,\"zero_default_value\":true,\"calculation_mode\":0,\"meta\":{},\"range_date\":false,\"display_html\":false,\"time_zone\":\"Asia/Manila\",\"script\":\"\\\"Hello World\\\"\"}"
}
```

Phát hiện:
- `meta: {}` — key bổ sung không có trong type def, thường rỗng.
- `time_zone` auto từ workspace setting.
- `script` lưu nguyên text user nhập (không parse).

### RULE-FIELD-FORMULA-07: Script syntax & helper functions

Editor hỗ trợ list functions theo nhóm:
- **Text**: `{text}.toLowerCase`, `.toUpperCase`, `.trim`, `.replace`, `.contains`, `.length`, `.substring`, `.startsWith`, `.endsWith`, `.isEmpty`, `.split`
- **Number**: `$number.format`, `.toNumber`, `.toInteger`, `.toLong`, `.toDouble`, `.round`, `.ceil`, `.floor`, `.abs`, `.min`, `.max`, `.strip`, `.stripGrouping`, `.stripChars`
- **Date**: `{date}.format`, `.addDays/Months/Years`, `.getYear/Month`, `.dayOfMonth/Year/Week`, `.diffDays`, `.isBefore/After`, `.isWeekend`
- **Datetime**: `{datetime}.format`, `.formatWithAccountTimeZone`, `.timestamp`, `.addDays/Hours/Minutes/Months`, `.getYear/Month/DayOfMonth/Hour/Minute/Second`, `.diffDays/Hours/Minutes`, `.isBefore/After`
- **Range**: `{dateRange|datetimeRange}.getStart`, `.getEnd`, `.range`, `.isInRange`
- **Util**: `$Util.toJsonArray`, `.escapeJson`, `.escapeHtml`, `.urlEncode`, `.join`, `.size`, `.sort`, `.today`, `.now`
- **SelectList**: `{selectList}.getValue`, `.getSelectedOption`, `.getSelectedOptions`
- **JSON**: `Json.parse`, `Json.stringify`

Toán tử: `+ - * / % () == != ! < <= > >= || &&`. Reference field qua `$record.<slug>`.

### RULE-FIELD-FORMULA-08: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`, `return_type` (đổi return_type cần migrate value), `calculation_mode`. Có thể sửa: `script`, `display_html`, `zero_default_value`. Sửa script ở mode `on-view` apply ngay, ở mode `on-write` cần re-trigger record update để re-compute.
