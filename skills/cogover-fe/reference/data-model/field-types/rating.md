---
title: Field Type — Xếp hạng (Rating)
impact: HIGH
impactDescription: rating_icons (alias DataFieldOption) lưu danh sách icon, tách khỏi meta_data; rating_type chuyển star ↔ reaction/emoji
tags: data-field, rating, RatingMetaData, ui-kit
---

## Field Type — Xếp hạng (Rating)

`fieldType = "rating"` (`DataTypes`). UI label: **"Xếp hạng (Rating)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/rating`.

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-RATING-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Kiểu hiển thị | `displayType` | `"Mặc định"` | Combobox 2 option (Mặc định/Tuỳ chỉnh) |
| 2 | Cách đánh giá | `ratingType` | `"Đánh giá theo thang điểm (Star rating)"` | Combobox 2 option: Star / Reaction |
| 3 | Số lượng biểu tượng | `displayNumber` | `5` (disabled khi default) | NumberInput |
| 4 | Giá trị mặc định | `defaultValue` | `0` | Click vào star để chọn 0-5 |
| 5 | Thiết lập biểu tượng | `ratingIcons` | `[{value: "star.svg", icon: ".../star.svg"}]` | Mặc định 1 star icon |

### RULE-FIELD-RATING-02: Input common bị ẩn

So với common (RULE-FIELD-COMMON-02):
- ❌ KHÔNG có Placeholder (`hint`)
- ❌ KHÔNG có "Cho phép sử dụng để Tìm kiếm nhanh" (`quickSearch`)
- ❌ KHÔNG có "Cho phép trường có nhiều giá trị" (`isMultipleValues`)
- ❌ KHÔNG có "Tính duy nhất" (`isUnique`)

### RULE-FIELD-RATING-03: Mapping form → `meta_data` (`RatingMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:615-629`.

```typescript
export interface RatingMetaData {
    rating_type?: RatingType;
    display_number?: number;
    display_type?: RatingDisplayType;
}

export enum RatingType { Star = 1, Reaction = 2 }
export enum RatingDisplayType { Default = 1, Custom = 2 }
```

| Form key | Đường dẫn trong `meta_data` |
|---|---|
| `displayType` | `display_type` (`Default = 1` / `Custom = 2`) |
| `ratingType` | `rating_type` (`Star = 1` / `Reaction = 2`) |
| `displayNumber` | `display_number` |

### RULE-FIELD-RATING-04: `ratingIcons` lưu RIÊNG (không trong meta_data)

`DataFieldRatingIcon = DataFieldOption` — top-level field của `DataField`, không nằm trong `fieldMetaData`. Khi gọi API create, truyền key `rating_icons` ở body level (top-level snake-case key, KHÔNG inside meta_data).

```typescript
export type DataFieldRatingIcon = DataFieldOption;
// CreateDataFieldParams có: rating_icons?: DataFieldRatingIcon[];
```

### RULE-FIELD-RATING-05: Payload thực tế (verify từ API)

Tạo "Test Rating Field" với defaults (Star, Mặc định, 5 sao):

```json
{
  "fieldType": "rating",
  "fieldMetaData": "{\"display_type\":1,\"rating_type\":1,\"display_number\":5}",
  "ratingIcons": [{
    "id": "OP0UCCQEVKM9F",
    "value": "star.svg",
    "icon": "/icon/dataTypes/fieldRating/star.svg",
    "sort": 1,
    "status": 1,
    "isDefault": 0,
    "slug": "1"
  }]
}
```

**Lưu ý**:
- `meta_data` có 3 key (`display_type`, `rating_type`, `display_number`) — `RatingMetaData` interface khớp.
- `ratingIcons` là **array riêng** (top-level), không nằm trong meta_data.
- Default `display_number = 5` (5 sao).

### RULE-FIELD-RATING-06: `rating_type` Star vs Reaction

- `Star = 1`: Render N icon giống nhau, click chọn 1-N.
- `Reaction = 2`: Render N icon **khác nhau** (vd 5 emoji 😡😞😐😊😍), click chọn 1, mỗi icon = 1 giá trị riêng.

Khi `Reaction`, `rating_icons` **bắt buộc** có N entries với `value` và `icon` khác nhau cho từng level.

### RULE-FIELD-RATING-07: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`. Có thể sửa: `display_type`, `rating_type`, `display_number`, `rating_icons`, `defaultValue`. Đổi `display_number` từ 5 → 3 không tự cap record cũ — record có rating > 3 vẫn lưu.
