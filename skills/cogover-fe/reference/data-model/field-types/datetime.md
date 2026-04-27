---
title: Field Type — Thời gian (Datetime, 7 variants)
impact: HIGH
impactDescription: 1 entry "Thời gian (Datetime)" trên UI map sang 7 fieldType khác nhau (date/time/date_time/duration + range); mỗi variant có metadata schema riêng
tags: data-field, date, time, date_time, time_duration, date_range, date_time_range, ui-kit
---

## Field Type — Thời gian (Datetime)

UI label: **"Thời gian (Datetime)"**. URL form tạo: `/settings/object/{objectSlug}/data-field/create/date-time` (slug `date-time`).

Trên UI là **1 entry** nhưng map sang **7 `fieldType` nội bộ** tuỳ combobox `displayType` + checkbox "Là khoảng thời gian":

| `displayType` | "Là khoảng thời gian" = false | = true |
|---|---|---|
| Ngày | `date` | `date_range` |
| Giờ | `time` | `time_range` |
| Thời lượng | `time_duration` | (verify — có thể không hỗ trợ range) |
| Ngày và giờ | `date_time` | `date_time_range` |

⇒ Tổng 7 `fieldType` enum: `date`, `time`, `time_duration`, `date_time`, `date_range`, `time_range`, `date_time_range`.

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-DATETIME-01: Form fields chung cho tất cả 7 variant

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Kiểu hiển thị | `displayType` | `"Ngày"` | Combobox 4 option |
| 2 | Định dạng hiển thị | `format` | `"Theo cấu hình chung của Workspace"` (`"workspace"`) | Combobox |
| 3 | Là khoảng thời gian | `isRange` | `false` | Checkbox |
| 4 | Cho phép trường có nhiều giá trị | `isMultipleValues` | `false` | Checkbox |
| 5 | Khoảng thời gian hợp lệ — Ngày nhỏ nhất / lớn nhất | `valueLimitFrom/To` | `null` | Date picker với mode (Ngày tuyệt đối / Tương đối) |
| 6 | Giá trị mặc định | `defaultValue` | (radio: "Ngày hiện tại" / "Chọn từ lịch") | Radio + Date picker |

### RULE-FIELD-DATETIME-02: Input common bị ẩn

So với common (RULE-FIELD-COMMON-02):
- ❌ KHÔNG có "Cho phép sử dụng để Tìm kiếm nhanh" (`quickSearch`)
- ❌ KHÔNG có "Tính duy nhất" (`isUnique`) — datetime trùng nhau là bình thường

### RULE-FIELD-DATETIME-03: Mapping form → `meta_data` theo từng variant

#### `date` / `time` → `DateMetaData` / `TimeMetaData`

```typescript
export type DateMetaData = Partial<{
    value_limit?: { from?: DateFieldValueLimitItem; to?: DateFieldValueLimitItem };
    multiple_limit?: { min?: number; max?: number };
    format?: { format?: string };
    default_value_current?: boolean;
}>;
```

`DateFieldValueLimitItem`:
```typescript
{
    type?: "absolute" | "relative-current" | "relative-to" | "relative-from";
    unit?: "day" | "week" | "month" | "year" | "hour" | "minute" | "second";
    value?: number | string;
}
```

#### `date_time` → `DateTimeMetaData`

```typescript
export type DateTimeMetaData = Partial<{
    date?: { value_limit?: {...} };
    format: { date: string; time: string };
    time?: { from?: number; to?: number };
    multiple_limit?: { min?: number; max?: number };
    default_value_current?: boolean;
}>;
```

#### `*_range` (date_range / time_range / date_time_range)

Có `start` và `end` thay vì single value, mỗi cái có `value_limit`:

```typescript
export type DateRangeMetaData = Partial<{
    start?: { value_limit?: {...} };
    end?: { value_limit?: {...} };
    format?: { format?: string };
}>;
```

#### `time_duration` → `DurationMetaData`

```typescript
export interface DurationMetaData {
    value_limit?: { from?: number | null; to?: number | null };
    multiple_limit?: { min?: number; max?: number };
    rules?: {
        week?: { unit?: string; code?: string; value?: number };
        // month, hour, year, day, minute, second, millisecond
    };
}
```

### RULE-FIELD-DATETIME-04: Payload thực tế (`date` variant)

Tạo "Test Datetime Field" với defaults (displayType=Ngày, không range, không multi):

```json
{
  "fieldType": "date",
  "fieldMetaData": "{\"value_limit\":{\"from\":null,\"to\":null},\"default_value_current\":false,\"format\":{\"format\":\"workspace\"},\"time_zone\":\"Asia/Manila\"}"
}
```

⚠️ **Phát hiện**:
- `format.format` lưu là **string** `"workspace"` (không phải `number` như type def `format?: number`).
- Có thêm key `time_zone` (vd `"Asia/Manila"`) **chưa có trong `DateMetaData`**. Bổ sung khi sync.

### RULE-FIELD-DATETIME-05: `default_value_current = true`

- `false` (default): Field rỗng khi tạo record mới, hoặc dùng `defaultValue` user nhập tay.
- `true`: Auto-fill thời gian **hiện tại** khi tạo record. Tương đương `now()`.

UI radio "Ngày hiện tại" / "Chọn từ lịch" map: "Ngày hiện tại" → `default_value_current: true`, "Chọn từ lịch" → `false` + `defaultValue: <timestamp>`.

### RULE-FIELD-DATETIME-06: `value_limit` mode "absolute" vs "relative"

Combobox "Ngày tuyệt đối" / Tương đối quyết định `type`:
- `"absolute"`: `value` là timestamp cố định (vd `1735689600000` = 1/1/2025)
- `"relative-current"`: relative tới `now()` (vd "trước 7 ngày")
- `"relative-from"` / `"relative-to"`: relative tới start/end của field

### RULE-FIELD-DATETIME-07: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`. Có thể sửa: `displayType` (đổi giữa các variant cùng nhóm — `date` ↔ `date_time`, nhưng range ↔ non-range thường KHÔNG cho phép vì DB schema khác). Có thể sửa: `format`, `value_limit`, `default_value`, `multiple_limit`.
