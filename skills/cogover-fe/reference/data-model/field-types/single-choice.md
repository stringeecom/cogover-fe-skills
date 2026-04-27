---
title: Field Type — Lựa chọn đơn (Single-choice / Radio button)
impact: HIGH
impactDescription: options array lưu RIÊNG khỏi meta_data; option_display_type chuyển icon/text/icon+text; isActiveTransitionRule chưa có trong type def
tags: data-field, single-choice, radio_button, SingleChoiceMetaData, ui-kit
---

## Field Type — Lựa chọn đơn (Single-choice / Radio button)

`fieldType = "single_choice"` (`DataTypes`). UI label: **"Lựa chọn đơn (Single-choice/ Radio button)"**.

`displayType` combobox quyết định render: dropdown (`single_choice`) hoặc radio button (`radio_button`). Cả 2 vẫn lưu `fieldType: "single_choice"` ở DB.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/single-choice`.

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-SINGLECHOICE-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Sử dụng lựa chọn toàn cục | `useGlobalPicklist` | `false` | Checkbox — dùng global picklist thay vì define options riêng |
| 2 | Kiểu hiển thị | `displayType` | `"Danh sách xổ xuống (Single-choice)"` | Combobox: Single-choice / Radio button |
| 3 | Sắp xếp các lựa chọn | `sortRule` | `"Như danh sách"` (None) | Radio: Như danh sách / A-Z / Z-A |
| 4 | Nội dung lựa chọn (table) | `options` | `[]` | Table với 8 cột: Lựa chọn, Biểu tượng, Slug, Hoạt động, Màu nền, Màu chữ, Mặc định, Phân loại trạng thái, Cấp trạng thái |
| 5 | Hiển thị | `optionDisplayType` | `"Chỉ nội dung văn bản"` | Combobox 3 option: IconText/Icon/Text |

### RULE-FIELD-SINGLECHOICE-02: Mapping form → `meta_data` (`SingleChoiceMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:358-362`.

```typescript
export interface SingleChoiceMetaData {
    sort_rule?: DataFieldOptionSortRule;
    prioritized_sort_rule?: DataFieldOptionPrioritizedSortRule;
    option_display_type?: SingleChoiceOptionDisplayType;
    // Bổ sung khi sync (verify từ payload):
    // isActiveTransitionRule?: boolean;  // ⚠️ camelCase, khác convention
}

export enum DataFieldOptionSortRule { None = 0, Asc = 1, Desc = 2 }
export enum DataFieldOptionPrioritizedSortRule { None = 0, Asc = 1, Desc = 2 }
export enum SingleChoiceOptionDisplayType { IconText = 0, Icon = 1, Text = 2 }
```

| Form key | Đường dẫn trong `meta_data` |
|---|---|
| `sortRule` | `sort_rule` |
| `optionDisplayType` (combobox "Hiển thị") | `option_display_type` |
| `useGlobalPicklist` | (không có trong meta — quản lý ở top-level / API riêng) |
| (UI ẩn) | `prioritized_sort_rule`, `isActiveTransitionRule` |

### RULE-FIELD-SINGLECHOICE-03: `options` lưu RIÊNG (không trong meta_data)

`options: DataFieldOption[]` là **top-level field** của `DataField`, không nằm trong `fieldMetaData`. Khi gọi API create, truyền `options` ở body level.

```typescript
export interface DataFieldOption {
    value?: string;       // Text hiển thị
    slug?: string;        // Slug định danh (auto-gen từ value)
    sort?: number;        // Thứ tự
    status?: number;      // 1 = active, 0 = inactive
    icon?: string | null;
    backgroundColor?: string;  // rgba/hex
    textColor?: string;
    isDefault?: number | null; // 1 = default option
    category?: string | null;     // Phân loại trạng thái (workflow status)
    category_level?: number;      // Cấp trạng thái
    // ... (xem dataField.type.ts:155-178 đầy đủ)
}
```

### RULE-FIELD-SINGLECHOICE-04: Payload thực tế (verify từ API)

Tạo "Test Single Choice Field" với 1 option "Option A":

```json
{
  "fieldType": "single_choice",
  "fieldMetaData": "{\"sort_rule\":0,\"isActiveTransitionRule\":false,\"prioritized_sort_rule\":0,\"option_display_type\":2}",
  "options": [{
    "id": "OPTPOTQEQKM95",
    "value": "Option A",
    "slug": "option_a",
    "sort": 1,
    "status": 1,
    "isDefault": 1,
    "icon": "",
    "backgroundColor": "rgba(0, 0, 0, 0)",
    "textColor": "#484848"
  }]
}
```

⚠️ **`isActiveTransitionRule` dùng camelCase** trong meta_data — khác hầu hết key snake_case. Type def chưa có key này.

`option_display_type: 2` = `Text` (chỉ text, không icon).

### RULE-FIELD-SINGLECHOICE-05: 3 option của `option_display_type`

| UI label | Enum value | Hiển thị record |
|---|---|---|
| Cả biểu tượng và văn bản | `IconText = 0` | `[icon] Option A` |
| Chỉ biểu tượng | `Icon = 1` | `[icon]` (tooltip = value) |
| Chỉ nội dung văn bản | `Text = 2` | `Option A` |

### RULE-FIELD-SINGLECHOICE-06: Khác biệt `single_choice` vs `radio_button`

- `single_choice`: dropdown XS (`<select>` style)
- `radio_button`: render N radio button bằng/cùng dòng

`fieldType` ở DB **luôn là** `"single_choice"`. UI sử dụng `displayType` trong meta hoặc field riêng để switch render.

Trong `DataTypes` enum có cả `radio_button` — có thể là legacy hoặc cho FE-side render hint.

### RULE-FIELD-SINGLECHOICE-07: `useGlobalPicklist`

Khi bật, options được lấy từ global picklist (workspace-level shared list) thay vì define cục bộ. UI sẽ thay table options bằng dropdown chọn picklist. Object link tới picklist qua field riêng (verify schema).

### RULE-FIELD-SINGLECHOICE-08: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`. Có thể sửa: thêm/sửa/xoá options, sort_rule, option_display_type. Đổi slug option có thể break filter/lookup nếu có record reference.
