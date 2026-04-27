---
title: Field Type — Tra cứu thường (Normal lookup)
impact: HIGH
impactDescription: Tự sinh related list trên target object; meta_data lưu object id + object_slug; cấu trúc filter qua slug.id (xem related-list)
tags: data-field, lookup, lookup_normal, LookupNormalMetaData, ui-kit, related-list
---

## Field Type — Tra cứu thường (Normal lookup)

`fieldType = "lookup_normal"` (`DataTypes`). UI label: **"Tra cứu thường (Normal lookup)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/lookup` (slug `lookup`, KHÔNG phải `lookup-normal`).

Cấu hình common: xem `field-types/common`. Liên hệ chéo: `data-model/related-list`.

### RULE-FIELD-LOOKUPNORMAL-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Tra cứu tới đối tượng | `object` | (chưa chọn — bắt buộc) | Combobox single-select object trong workspace |
| 2 | Tên danh sách liên quan | `relatedListName` | `<source object name>` (auto) | Textbox **bắt buộc** — tên related list tự sinh trên target |
| 3 | Slug của danh sách liên quan | `relatedListSlug` | `<source object slug>` (auto) | Textbox **bắt buộc** — slug related list |
| 4 | Giá trị mặc định | `defaultValue` | (không có UI) | Lookup không hỗ trợ default value |

### RULE-FIELD-LOOKUPNORMAL-02: Input common bị ẩn

So với common (RULE-FIELD-COMMON-02):
- ❌ KHÔNG có "Giá trị mặc định" (`defaultValue`)

### RULE-FIELD-LOOKUPNORMAL-03: Mapping form → `meta_data` (`LookupNormalMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:589-591`.

```typescript
export interface LookupNormalMetaData extends LookupMetaData {
    link_field: string;
}

export interface LookupMetaData {
    object?: string;              // ID của target object
    object_slug?: string;         // Slug của target object
    multiple_limit?: { min?: number; max?: number };
    unique_rule?: number;
    lookup_filter?: string;       // JSON filter
    lookup_filter_error_msg?: string;
    lookup_filter_status?: boolean;
    lookup_filter_type?: LookupFilterType;
    related_list_filter?: string;
    related_list_filter_related_list_id?: string;
    related_list_filter_status?: boolean;
}
```

| Form key | Đường dẫn |
|---|---|
| `object` (combobox) | `meta_data.object` (id) + `meta_data.object_slug` |
| `relatedListName` | top-level `related_list_name` |
| `relatedListSlug` | top-level `related_list_slug` |
| (UI ẩn) | `meta_data.link_field` (rỗng default) |
| (UI ẩn) | `meta_data.lookup_filter`, `lookup_filter_status`, `related_list_filter` (config nâng cao) |

### RULE-FIELD-LOOKUPNORMAL-04: Payload thực tế (verify từ API)

Tạo "Test Lookup Field" tra cứu tới object `user`:

```json
POST /api/v1/objects/field/create
{
  "type": "lookup_normal",
  "object_type_id": "OTTPOTQETKMSA",
  "workspace_id": "WSsgfBnZoNrPA",
  "name": "Test Lookup Field",
  "slug": "test_lookup_field",
  "related_list_name": "Test ShortText Obj",
  "related_list_slug": "test_shorttext_obj_lookup",
  "meta_data": "{\"object\":\"OTGVJ2QEEKRPB\",\"object_slug\":\"user\",\"multiple_limit\":{\"min\":0,\"max\":30},\"link_field\":\"\"}"
}
```

Response field:

```json
{
  "fieldType": "lookup_normal",
  "fieldMetaData": "{\"object\":\"OTGVJ2QEEKRPB\",\"object_slug\":\"user\",\"multiple_limit\":{\"min\":0,\"max\":30},\"link_field\":\"\"}",
  "relatedList": {
    "id": "RL0UCCQEZKM9N",
    "name": "Test ShortText Obj",
    "slug": "test_shorttext_obj_lookup",
    "lookupType": "lookup_normal",
    "sourceObjectId": "OTTPOTQETKMSA",
    "sourceObjectSlug": "test_shorttext_obj",
    "sourceFieldId": "OF0UCCQE1KM9R",
    "sourceFieldSlug": "test_lookup_field",
    "lookupObjectId": "OTGVJ2QEEKRPB",
    "lookupObjectSlug": "user"
  }
}
```

### RULE-FIELD-LOOKUPNORMAL-05: Related list tự sinh trên target

Tạo lookup field trong A trỏ sang B → BE **tự sinh related list** trên B với:
- `sourceObjectSlug = A.slug`
- `sourceFieldSlug = <field-slug-vừa-tạo>`
- `lookupObjectSlug = B.slug`

Xem `data-model/related-list` để query bản ghi liên quan.

### RULE-FIELD-LOOKUPNORMAL-06: Filter field path khi query

Khi filter record qua field lookup_normal:
- Field type `lookup_normal` → filter `field` = `<sourceFieldSlug>` (vd `"test_lookup_field"`)
- Field type `reference` → filter `field` = `<sourceFieldSlug>.id` (xem RULE-RL-04)

```typescript
const filters: FilterGeneratorItem[] = [{
  field: 'test_lookup_field',  // lookup_normal: just slug
  op: '=',
  params: 'PERUO1MQTEKXXA',     // record id
}];
```

### RULE-FIELD-LOOKUPNORMAL-07: `link_field` (advanced)

`link_field` (string) là slug của 1 field trên target object — khi có giá trị, FE render display dựa trên field này thay vì `name` mặc định. Vd lookup tới User, set `link_field = "email"` → hiển thị email thay vì full_name.

Default `""` = dùng `name` field của target.

### RULE-FIELD-LOOKUPNORMAL-08: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`, `object` (target object), `related_list_slug`. Có thể sửa: `relatedListName`, `link_field`, lookup_filter (nâng cao), `multiple_limit`.

Xoá field lookup → BE tự xoá related list tương ứng trên target.
