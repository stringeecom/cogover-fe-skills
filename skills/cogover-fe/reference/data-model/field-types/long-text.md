---
title: Field Type — Văn bản dài (Long-text)
impact: HIGH
impactDescription: Sai cấu hình text_types (Plain/RichText/Markdown) làm record render lệch editor; max_length 131072 đặc thù cho long-text
tags: data-field, long-text, long_text, LongTextMetaData, ui-kit
---

## Field Type — Văn bản dài (Long-text)

`fieldType = "long_text"` (`DataTypes`). UI label: **"Văn bản dài (Long-text)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/long-text`.

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-LONGTEXT-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Kiểu soạn thảo văn bản — Văn bản thuần túy (Plain text) | `textTypes[0]` | `true` | Checkbox, mapping `LongTextType.Normal = 1` |
| 2 | Kiểu soạn thảo văn bản — Trình soạn thảo văn bản (WYSIWYG) | `textTypes[1]` | `false` | Checkbox, mapping `LongTextType.RichText = 2` |
| 3 | Kiểu soạn thảo văn bản — Trình soạn thảo hỗ trợ Markdown | `textTypes[2]` | `false` | Checkbox, mapping `LongTextType.Markdown = 3` |
| 4 | Số lượng ký tự — Tối thiểu | `minLength` | `0` | NumberInput |
| 5 | Số lượng ký tự — Tối đa | `maxLength` | `131072` | NumberInput, **giới hạn cứng 131072** (đặc thù long-text, ~128 KiB) |
| 6 | Giá trị mặc định | `defaultValue` | `""` | Textarea multiline. **Khi chọn nhiều text_types**, có 3 tab default value riêng (Văn bản thuần / Văn bản có định dạng / Markdown) |

### RULE-FIELD-LONGTEXT-02: Input common bị ẩn / khoá

So với common (RULE-FIELD-COMMON-02):
- ❌ KHÔNG có "Cho phép trường có nhiều giá trị" (`isMultipleValues`) — long-text **chỉ single value**
- ❌ KHÔNG có "Tính duy nhất" (`isUnique`)
- ❌ KHÔNG có combobox "Kiểu hiển thị" (đã có 3 checkbox text_types thay)

### RULE-FIELD-LONGTEXT-03: Multi-select 3 kiểu soạn thảo

3 checkbox **không phải radio** — user có thể bật **nhiều** type cùng lúc. Khi bật nhiều, record sẽ có dropdown chọn editor mode khi nhập.

| UI label | Enum (`LongTextType`) |
|---|---|
| Văn bản thuần túy (Plain text) | `Normal = 1` |
| Trình soạn thảo văn bản (WYSIWYG) | `RichText = 2` |
| Trình soạn thảo hỗ trợ Markdown | `Markdown = 3` |

**Validate**: phải chọn ít nhất 1 (không cho uncheck cả 3).

### RULE-FIELD-LONGTEXT-04: Mapping form → `meta_data` (`LongTextMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:243-249`.

```typescript
export interface LongTextMetaData {
    character_limit?: { min?: number; max?: number };
    text_type?: LongTextType;       // primary = phần tử đầu tiên trong text_types
    text_types?: LongTextType[];    // danh sách bật
    default_values?: Record<string, string>; // map text_type → default text riêng
    rich_text?: boolean;            // legacy, KHÔNG dùng cho field mới
}
```

| Form key | Đường dẫn trong `meta_data` | Ghi chú |
|---|---|---|
| `minLength` | `character_limit.min` | |
| `maxLength` | `character_limit.max` | Max 131072 |
| Checkbox text_types đã chọn (sorted) | `text_types: number[]` | Mảng giá trị enum |
| Phần tử đầu tiên của `text_types` | `text_type: number` | BE auto-set = `text_types[0]` |
| `defaultValue` (theo từng tab) | `default_values: { "1": "...", "2": "...", "3": "..." }` | Object **string-key** = enum |

### RULE-FIELD-LONGTEXT-05: Payload thực tế (verify từ API response)

Tạo field "Test Long Text Field" với cả 3 text types (Plain + WYSIWYG + Markdown), không default value:

**Stored `fieldMetaData`** (sau khi GET object detail với `withFields=1`):

```json
{
  "character_limit": { "min": 0, "max": 131072 },
  "text_type": 1,
  "text_types": [1, 2, 3],
  "default_values": {}
}
```

**Field record trả về**:

```json
{
  "id": "OFTPOTQEIKM90",
  "slug": "test_long_text_field",
  "name": "Test Long Text Field",
  "fieldType": "long_text",
  "fieldMetaData": "<JSON string ở trên>",
  "required": 0,
  "multiple": 0,
  "unique": false,
  "isStandard": 0,
  "manualModifyAllow": true
}
```

### RULE-FIELD-LONGTEXT-06: `default_values` map theo enum string-key

Khác với `defaultValue` ở field text khác (string thuần), long-text dùng object map:

```json
{
  "default_values": {
    "1": "Default plain text",
    "2": "<p>Default <b>HTML</b></p>",
    "3": "# Default markdown"
  }
}
```

Key là **string của enum number** (`"1"`, `"2"`, `"3"`). Khi user xem record, FE đọc theo `text_type` hiện tại để hiển thị default tương ứng.

### RULE-FIELD-LONGTEXT-07: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`. Có thể sửa: tất cả còn lại bao gồm `text_types` (thêm/bớt mode), `character_limit`, `default_values`.

### RULE-FIELD-LONGTEXT-08: Khác biệt với `short_text` display=MultipleLines

Cả 2 đều render textarea, **nhưng**:
- `short_text` + `display_type=MultipleLines` → vẫn lưu DB là VARCHAR/short, max 255 mặc định, không hỗ trợ rich text/markdown.
- `long_text` → lưu DB là TEXT/long, max **131072**, hỗ trợ Plain + RichText + Markdown đa chế độ.
- Chọn `long_text` khi user cần nhập nhiều/định dạng phong phú; chọn `short_text` khi chỉ cần text ngắn vài câu.
