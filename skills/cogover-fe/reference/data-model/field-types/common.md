---
title: Cấu hình chung của các Field Type
impact: HIGH
impactDescription: Sai mapping form field ↔ CreateDataFieldParams gây tạo field thiếu thuộc tính / sai schema
tags: data-field, create-field, common-config, meta-data, ui-kit
---

## Cấu hình chung của các Field Type

Trang `Settings → Object → {object} → Cấu hình trường → Tạo trường → Bước 2 (Cấu hình trường)` hiển thị 1 form 2 vùng:
- **Vùng cấu hình** (trái) — phần dưới đây
- **Vùng "Xem trước"** (phải) — render preview field theo cấu hình

Form sử dụng `react-hook-form` + Controlled components từ `@stringeecom/ui-kit`. **Tất cả** field type đều có **các trường common** dưới đây; sau đó tuỳ type sẽ thêm các trường riêng (xem từng file `<type>.md` trong cùng thư mục).

### RULE-FIELD-COMMON-01: Hiển thị tóm tắt kiểu dữ liệu

Trên đầu form luôn có dòng đọc-only:

```
Kiểu dữ liệu: <Tên hiển thị của type>
```

Đây là field user đã chọn ở Bước 1, **không thể đổi sau khi đã tạo field** — sửa field chỉ cho đổi metadata, không đổi `fieldType`.

### RULE-FIELD-COMMON-02: Form fields chung (12 input)

| # | UI label | Form name (`react-hook-form`) | Required | Default | Mô tả |
|---|---|---|---|---|---|
| 1 | Tên trường | `name` | ✅ | `""` | Tên hiển thị của field |
| 2 | Slug | `slug` | ✅ | tự gen từ `name` (snake_case) | Định danh duy nhất, **không sửa được sau khi tạo** |
| 3 | Nội dung hướng dẫn (tooltip) | `tooltip` | — | `""` | Tooltip cạnh tên field, **max 255 ký tự** |
| 4 | Placeholder | `hint` | — | `""` | Placeholder bên trong input, **max 255 ký tự** |
| 5 | Mô tả trường | `description` | — | `""` | Mô tả nội bộ, **max 1000 ký tự**, textarea multiline |
| 6 | Trường bắt buộc | `isRequired` | — | `false` | Checkbox — bắt buộc nhập khi tạo/sửa record |
| 7 | Cho phép tạo, sửa thủ công | `allowManualEdit` | — | `true` | Checkbox — cho phép user nhập tay (vs read-only / system) |
| 8 | Trạng thái Đang hoạt động | `isActive` | — | `true` | Checkbox — `DataFieldStatus.Active = 1` / `Inactive = 0` |
| 9 | Cho phép sử dụng để Tìm kiếm nhanh | `quickSearch` | — | `false` | Checkbox — bật trong filter quick-search của list view |
| 10 | Cho phép trường có nhiều giá trị | `isMultipleValues` | — | `false` | Checkbox — multi value, mở UI cấu hình `multiple_limit` |
| 11 | Tính duy nhất | `isUnique` | — | `false` | Checkbox — không cho trùng giá trị giữa các record |
| 12 | Giá trị mặc định | `defaultValue` | — | `""` (kiểu phụ thuộc type) | Giá trị tự điền khi tạo record mới |

> **Lưu ý**: Một số type **không có** đủ 12 input này (vd: `formula` không có `defaultValue`/`isRequired`; `boolean` ép `isMultipleValues = false`...). Mỗi file type sẽ liệt kê **input nào bị ẩn / khoá**.

### RULE-FIELD-COMMON-03: Mapping form → API body

**Endpoint**: `POST /api/v1/objects/field/create` (đã verify từ network — KHÔNG phải `/data-field/create` như interface name gợi ý).

Body **mix snake_case + camelCase** (theo backend API hiện tại):

```typescript
import type { CreateDataFieldParams } from "@stringeecom/ui-kit/apis/dataField/dataField.type";
```

| Form key | API key | Kiểu giá trị | Convention |
|---|---|---|---|
| `name` | `name` | `string` | snake |
| `slug` | `slug` | `string` | snake |
| `tooltip` | `tool_tip` | `string` | snake |
| `hint` | `hint_text` | `string` | snake |
| `description` | `description` | `string` | snake |
| `isRequired` | `required` | `0 \| 1` | snake |
| `allowManualEdit` | `manual_modify_allow` | `boolean` | snake |
| `isActive` | `status` | `0 \| 1` (`DataFieldStatus`) | snake |
| `quickSearch` | `quickSearch` | `0 \| 1` | **camelCase** ⚠️ |
| `isMultipleValues` | `multiple` | `0 \| 1` | snake |
| `isUnique` | `unique` | `0 \| 1` | snake |
| `defaultValue` | `default_value` | `string \| boolean \| number \| null \| { gte; lte }` | snake |

**Bắt buộc luôn truyền**:
- `object_type_id` — id của object đang tạo field (lấy từ `object.id`, vd `"OTTPOTQETKMSA"`)
- `workspace_id` — id workspace hiện tại (vd `"WSsgfBnZoNrPA"`)
- `type` — `DataTypes` enum (vd: `"short_text"`, `"numeric"`)
- `meta_data` — **JSON.stringify** của metadata theo `<type>MetaData` interface tương ứng

**Response envelope**:

```json
{
  "tags": [],
  "r": 0,
  "msg": "OK",
  "requestId": "RQ<uuid>",
  "data": [{ "id": "OFT<...>", "slug": "<slug>", "options": [] }],
  "meta": null
}
```

`r === 0` = thành công. `data` là **array** (luôn 1 phần tử khi tạo 1 field).

### RULE-FIELD-COMMON-04: `multiple` vs `multiple_limit`

2 chỗ — **không trùng nghĩa**:
- `multiple` (top-level, `0 \| 1`) = **cờ bật** chế độ nhiều giá trị (checkbox "Cho phép trường có nhiều giá trị")
- `multiple_limit: { min, max }` (trong `meta_data`) = **giới hạn số lượng** khi `multiple = 1`

**Hành vi thực tế** (verify từ payload):
- Khi `isMultipleValues = false` (form) → API vẫn gửi `multiple_limit: { min: 1, max: 30 }` + `multiple: 0`. **Max=30 là default cap** của FE, BE bỏ qua khi `multiple=0`.
- Khi `isMultipleValues = true` → mở UI nhập `min`/`max` (default `min=1`, `max=30`), gửi đúng giá trị user nhập.

Tóm lại: luôn gửi `multiple_limit` trong meta_data, đừng phụ thuộc giá trị này khi `multiple=0`.

### RULE-FIELD-COMMON-05: `isUnique` vs `unique_rule`

Tương tự, `unique` ở top-level (`CreateDataFieldParams.unique`) là cờ bật. Một số metadata còn có `unique_rule?: number` (vd: `ShortTextMetaData`, `EmailMetaData`, `PhoneNumberMetaData`...) chứa rule chi tiết. Mặc định, FE chỉ set `unique` ở top-level và bỏ qua `unique_rule` trừ khi UI có cấu hình rule riêng.

### RULE-FIELD-COMMON-06: Type của `defaultValue` phụ thuộc field type

| Field type | Kiểu `defaultValue` |
|---|---|
| `short_text`, `long_text`, `email`, `url`, `phone`, `regex`, `label` | `string` |
| `numeric`, `decimal`, `percent`, `currency`, `auto_number`, `rating` | `number` |
| `boolean`, `checkbox` | `boolean` |
| `date`, `date_time`, `time`, `time_duration` | `number` (timestamp/duration) **hoặc** `null` |
| `*_range` (date_range, time_range, date_time_range) | `{ gte: number \| null; lte: number \| null }` |
| `single_choice`, `radio_button` | `string` (slug option) |
| `multi_choices`, `checkbox` (nhóm) | `string` (slug joined) hoặc `null` |
| `lookup_*`, `reference`, `formula`, `rollup_summary` | `null` (không hỗ trợ default value) |

### RULE-FIELD-COMMON-07: Slug auto-generate

Khi user nhập `name`, FE tự generate `slug` theo rule:
1. Lower case
2. Bỏ dấu (Vietnamese diacritics → ASCII)
3. Thay khoảng trắng + ký tự đặc biệt → `_`
4. Cắt prefix/suffix `_` thừa

User vẫn sửa được `slug` thủ công trước khi submit. **Sau khi tạo**, slug **không sửa được**.

### RULE-FIELD-COMMON-08: Quy tắc Validate trước khi submit

- `name` — không rỗng, không quá 255 ký tự
- `slug` — không rỗng, regex `/^[a-z][a-z0-9_]*$/`, không trùng với field khác cùng object
- `tooltip`, `hint` — max 255
- `description` — max 1000
- `multiple_limit.min ≤ multiple_limit.max`
- `character_limit.min ≤ character_limit.max` (cho text type)
- `value_limit.min ≤ value_limit.max` (cho number/percent/currency)

Nút "Hoàn thành" disable khi có error validate.

### RULE-FIELD-COMMON-09: Bước nav giữa 2 step

- "Quay lại" — về Bước 1 (chọn type), giữ data đã nhập trong Bước 2 nếu cùng type
- "Hủy" — quay về `/settings/object/{slug}/data-field` (list field)
- "Hoàn thành" — submit. Sau khi success → redirect về list field, hiện toast "Tạo trường thành công"
