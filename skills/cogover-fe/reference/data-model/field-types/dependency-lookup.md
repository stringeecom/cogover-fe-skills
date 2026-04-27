---
title: Field Type — Tra cứu phụ thuộc (Dependency lookup)
impact: HIGH
impactDescription: Master-detail relationship, isRequired luôn true; link_field bắt buộc và tự sinh field tương ứng trên target object; rollup_summary phụ thuộc vào field này
tags: data-field, lookup, lookup_parent, dependency-lookup, LookupMetaData, ui-kit
---

## Field Type — Tra cứu phụ thuộc (Dependency lookup)

`fieldType = "lookup_parent"` (`DataTypes`). UI label: **"Tra cứu phụ thuộc (Dependency lookup)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/lookup?lookup_type=dependency` (cùng URL với `lookup_normal`, phân biệt qua query param `lookup_type=dependency`).

Cấu hình common: xem `field-types/common`. Liên hệ chéo: `data-model/related-list`, `field-types/lookup-normal`, `field-types/rollup-summary`.

### RULE-FIELD-DEPENDENCYLOOKUP-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Tra cứu tới đối tượng | `object` | (chưa chọn — bắt buộc) | Combobox single-select |
| 2 | (UI ẩn / set ngầm) | `linkField` | (slug được auto-gen từ field này) | Field slug tự sinh trên target object |
| 3 | Tên danh sách liên quan | `relatedListName` | (auto) | **bắt buộc** |
| 4 | Slug của danh sách liên quan | `relatedListSlug` | (auto) | **bắt buộc** |

### RULE-FIELD-DEPENDENCYLOOKUP-02: Input common bị ẩn / khoá

So với common (RULE-FIELD-COMMON-02):
- ⚠️ "Trường bắt buộc" (`isRequired`) → **luôn `true` và disabled** — master-detail bắt buộc child có parent
- ❌ KHÔNG có "Cho phép trường có nhiều giá trị" (`isMultipleValues`) — luôn single (`multiple_limit: { min: 1, max: 1 }`)
- ❌ KHÔNG có "Tính duy nhất" (`isUnique`)
- ❌ KHÔNG có "Giá trị mặc định" (`defaultValue`)

### RULE-FIELD-DEPENDENCYLOOKUP-03: `link_field` bắt buộc và unique trên target

`link_field` là **slug của field "ngược"** sẽ được tự sinh trên **target object**. Vd dependency từ A → B với `link_field: "parent_a"` → object B sẽ có field `parent_a` lookup ngược về A.

**Validate**:
- `link_field` không được trùng với slug field hiện có trên target → BE trả lỗi `linkField "<slug>" in object "<id>" existed`.
- BE bắt buộc `link_field` không rỗng → lỗi `object or link_field is empty` nếu thiếu.

### RULE-FIELD-DEPENDENCYLOOKUP-04: Mapping form → `meta_data` (`LookupMetaData`)

```typescript
// LookupMetaData (cùng struct với lookup_normal nhưng KHÔNG có link_field theo type def)
export interface LookupMetaData {
    object?: string;
    object_slug?: string;
    multiple_limit?: { min?: number; max?: number };
    unique_rule?: number;
    lookup_filter?: string;
    lookup_filter_status?: boolean;
    lookup_filter_type?: LookupFilterType;
    related_list_filter?: string;
    related_list_filter_status?: boolean;
}
```

⚠️ Trong type def, `LookupNormalMetaData extends LookupMetaData` mới có `link_field`. Nhưng thực tế `lookup_parent` cũng dùng `link_field` — type def có thể outdated, cần sync.

### RULE-FIELD-DEPENDENCYLOOKUP-05: Payload thực tế (verify từ API)

Tạo "Test Dependency Field" tra cứu sang `user`, link_field=`test_dep_link_field`:

```json
POST /api/v1/objects/field/create
{
  "type": "lookup_parent",
  "object_type_id": "OTTPOTQETKMSA",
  "workspace_id": "WSsgfBnZoNrPA",
  "name": "Test Dependency Field",
  "slug": "test_dependency_field",
  "required": 1,
  "multiple": 0,
  "unique": 0,
  "related_list_name": "Test ShortText Obj DepLookup",
  "related_list_slug": "test_shorttext_obj_dep",
  "meta_data": "{\"object\":\"OTGVJ2QEEKRPB\",\"object_slug\":\"user\",\"multiple_limit\":{\"min\":1,\"max\":1},\"link_field\":\"test_dep_link_field\"}"
}
```

Stored:

```json
{
  "fieldType": "lookup_parent",
  "fieldMetaData": "{\"object\":\"OTGVJ2QEEKRPB\",\"object_slug\":\"user\",\"multiple_limit\":{\"min\":1,\"max\":1},\"link_field\":\"test_dep_link_field\"}"
}
```

### RULE-FIELD-DEPENDENCYLOOKUP-06: Khác biệt với `lookup_normal`

| Khía cạnh | `lookup_normal` | `lookup_parent` (dependency) |
|---|---|---|
| Quan hệ | Loose reference | **Master-detail** ràng buộc |
| `isRequired` | Tuỳ chọn | **Luôn true** (forced) |
| `multiple` | 0 hoặc 1 | **Luôn 0** (single parent) |
| `link_field` | Tuỳ chọn (display field hint) | **Bắt buộc** (tạo field ngược) |
| Xoá parent record | Child không bị xoá | Child **bị cascade delete** (verify) |
| `rollup_summary` | KHÔNG hỗ trợ | **Hỗ trợ** — child rollup → parent |

### RULE-FIELD-DEPENDENCYLOOKUP-07: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`, `object`, `link_field`, `related_list_slug`. Có thể sửa: `relatedListName`, `lookup_filter` (nâng cao). Xoá field dependency → cascade xoá field ngược (`link_field`) trên target và **tất cả record con liên quan**.
