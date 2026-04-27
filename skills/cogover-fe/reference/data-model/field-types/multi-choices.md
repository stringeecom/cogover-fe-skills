---
title: Field Type — Lựa chọn nhiều (Multi-choices / Checkboxes)
impact: HIGH
impactDescription: object_picker biến options thành record picker từ object khác; multiple_limit giới hạn số lựa chọn
tags: data-field, multi-choices, multi_choices, checkbox, MultipleChoiceMetaData, ui-kit
---

## Field Type — Lựa chọn nhiều (Multi-choices / Checkboxes)

`fieldType = "multi_choices"` (`DataTypes`). UI label: **"Lựa chọn nhiều (Multi-choices/ Checkboxes)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/multiple-choice` ⚠️ (slug `multiple-choice`, KHÔNG phải `multi-choices` hay `multi-choice`).

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-MULTICHOICES-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Sử dụng lựa chọn toàn cục | `useGlobalPicklist` | `false` | Checkbox |
| 2 | Số lượng lựa chọn — Tối thiểu | `multipleLimitMin` | `0` (disabled khi không required) | NumberInput |
| 3 | Số lượng lựa chọn — Tối đa | `multipleLimitMax` | `(empty)` / `30` | NumberInput |
| 4 | Kiểu hiển thị | `displayType` | `"Danh sách xổ xuống (Multi-choice)"` | Combobox: Multi-choice / Checkboxes |
| 5 | Sắp xếp các lựa chọn | `sortRule` | `"Như danh sách"` | Radio (giống single-choice) |
| 6 | Dùng danh sách đối tượng làm danh sách lựa chọn | `objectPicker` | `false` | Checkbox — bật → options lấy từ object khác |
| 7 | Nội dung lựa chọn (table) | `options` | `[]` | Table (giống single-choice nhưng KHÔNG có category/category_level) |

### RULE-FIELD-MULTICHOICES-02: Input common bị ẩn

So với common (RULE-FIELD-COMMON-02):
- ❌ KHÔNG có "Cho phép sử dụng để Tìm kiếm nhanh" (`quickSearch`)
- ❌ KHÔNG có "Cho phép trường có nhiều giá trị" — luôn multi (true)

### RULE-FIELD-MULTICHOICES-03: Mapping form → `meta_data` (`MultipleChoiceMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:265-272`.

```typescript
export interface MultipleChoiceMetaData extends SingleChoiceMetaData {
    multiple_limit?: { min?: number; max?: number };
    unique_rule?: number;
    object_picker?: boolean;
}
```

Kế thừa từ `SingleChoiceMetaData` (`sort_rule`, `prioritized_sort_rule`, `option_display_type`).

| Form key | Đường dẫn trong `meta_data` |
|---|---|
| `multipleLimitMin` / `multipleLimitMax` | `multiple_limit.min/max` |
| `objectPicker` | `object_picker` |
| `sortRule` | `sort_rule` |
| (UI ẩn) | `prioritized_sort_rule` |

### RULE-FIELD-MULTICHOICES-04: Payload thực tế (verify từ API)

Tạo "Test Multi Choice Field" với 1 option "Choice 1":

```json
{
  "fieldType": "multi_choices",
  "fieldMetaData": "{\"sort_rule\":0,\"prioritized_sort_rule\":0,\"object_picker\":false,\"multiple_limit\":{\"min\":0,\"max\":30}}",
  "options": [{
    "id": "OP0UCCQEWKM9G",
    "value": "Choice 1",
    "slug": "choice_1",
    "sort": 1,
    "status": 1,
    "isDefault": 1,
    "backgroundColor": "rgba(0, 0, 0, 0)",
    "textColor": "#484848"
  }]
}
```

⚠️ Khác `single_choice`:
- KHÔNG có `option_display_type` trong meta (multi-choice không có icon mode)
- KHÔNG có `isActiveTransitionRule` (chỉ single-choice có)
- THÊM `multiple_limit` và `object_picker`

### RULE-FIELD-MULTICHOICES-05: `object_picker = true` mode

Khi bật, FE chuyển từ table options sang dropdown chọn object (target object). Options khi đó **không** được lưu trong `options` array — record sẽ chứa list `record_id` của target object thay vì option slug.

Tương tự `lookup_normal` nhưng nhẹ hơn (không có lookup filter, không tự sinh related list).

### RULE-FIELD-MULTICHOICES-06: Khác biệt `multi_choices` vs `checkbox`

- `multi_choices`: dropdown multi-select XS
- `checkbox`: render N checkbox dạng list

`fieldType` luôn `"multi_choices"`. UI dùng `displayType` để switch render.

### RULE-FIELD-MULTICHOICES-07: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`, `object_picker` (nếu đã bật, không tắt được; verify). Có thể sửa: thêm/bớt options, multiple_limit, sort_rule.
