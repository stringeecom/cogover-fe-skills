---
title: Field Type — Tra cứu phụ thuộc (Dependency lookup)
impact: HIGH
impactDescription: fieldType "reference" + meta_data.link_field "id" để register dependency parent-child. isRequired forced true; multiple = 0; rollup_summary phụ thuộc field này
tags: data-field, lookup, reference, dependency-lookup, ReferenceMetaData, ui-kit
---

## Field Type — Tra cứu phụ thuộc (Dependency lookup)

`fieldType = "reference"`. UI label: **"Tra cứu phụ thuộc (Dependency lookup)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/lookup?lookup_type=dependency` (cùng URL với `lookup_normal`, phân biệt qua query param `lookup_type=dependency`).

Cấu hình common: xem `field-types/common`. Liên hệ chéo: `data-model/related-list`, `field-types/lookup-normal`, `field-types/rollup-summary`.

### RULE-FIELD-DEPENDENCYLOOKUP-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Tra cứu tới đối tượng | `object` | (chưa chọn — bắt buộc) | Combobox single-select target object |
| 2 | (UI ẩn) | `link_field` | `"id"` (auto, hardcoded) | Luôn `"id"` cho dependency — khác với `lookup_normal` (`""`) |
| 3 | Tên danh sách liên quan | `relatedListName` | `<source object name>` (auto) | **bắt buộc** — top-level field |
| 4 | Slug của danh sách liên quan | `relatedListSlug` | `<source object slug>` (auto) | **bắt buộc** — top-level field |
| 5 | Trường bắt buộc | `required` | `1` (forced, disabled) | Luôn checked, không sửa được |

### RULE-FIELD-DEPENDENCYLOOKUP-02: Input common bị ẩn / khoá

So với common (RULE-FIELD-COMMON-02):
- ⚠️ "Trường bắt buộc" (`required`) → **luôn `1` và disabled** — master-detail bắt buộc child có parent
- ❌ KHÔNG có "Cho phép trường có nhiều giá trị" (`multiple`) — luôn single (`0`)
- ❌ KHÔNG có "Tính duy nhất" (`unique`)
- ❌ KHÔNG có "Giá trị mặc định" (`defaultValue`)

### RULE-FIELD-DEPENDENCYLOOKUP-03: `link_field = "id"` cố định

Khác `lookup_normal` (link_field tuỳ chọn — slug của field hiển thị thay `name`):
- `lookup_normal`: `link_field` mặc định `""`, có thể set sang slug field bất kỳ trên target
- `reference` (dependency): `link_field` **luôn `"id"`** — BE dùng cờ này để nhận dạng đây là dependency parent-child, KHÔNG phải display hint

Form UI không hiển thị input cho `link_field` (FE auto-set `"id"` khi user chọn "Tra cứu phụ thuộc" trong combobox data type).

### RULE-FIELD-DEPENDENCYLOOKUP-04: Mapping form → `meta_data` (`ReferenceMetaData`)

```typescript
// Theo payload thực tế quan sát từ form (sync với type def cần thêm 'reference' variant)
export interface ReferenceMetaData {
    object?: string;                        // Target object id (parent)
    object_slug?: string;                   // Target object slug
    link_field: "id";                       // Cố định "id" cho dependency
    lookup_filter?: string;                 // ID filter (FIxxx) khi save filter
    lookup_filter_status?: boolean;
    lookup_filter_type?: LookupFilterType;  // 1=Required, 2=None
    lookup_filter_error_msg?: string;       // Tiếng Việt vd "Object 1 không tồn tại hoặc không khớp với tiêu chí lọc."
    related_list_filter_related_list_id?: string | null;
    related_list_filter_status?: boolean;
    allow_searching?: 0 | 1;                // 0 default
}
```

### RULE-FIELD-DEPENDENCYLOOKUP-05: Payload thực tế (verify từ API)

Tạo "Test Dep" tra cứu phụ thuộc sang `User`:

```json
POST /api/v1/objects/field/create
{
  "type": "reference",
  "object_type_id": "<source-id>",
  "workspace_id": "<ws-id>",
  "name": "Test Dep",
  "slug": "test_dep",
  "tool_tip": "",
  "default_value": null,
  "required": 1,
  "description": "",
  "hint_text": "",
  "status": 1,
  "manual_modify_allow": true,
  "multiple": 0,
  "unique": 0,
  "quickSearch": 0,
  "related_list_name": "Test ShortText Obj",
  "related_list_slug": "test_shorttext_obj",
  "lookup_filter": {
    "logicType": "AND",
    "logic": "",
    "conditions": [],
    "sortFields": []
  },
  "meta_data": "{\"object\":\"<target-id>\",\"link_field\":\"id\",\"object_slug\":\"<target-slug>\",\"lookup_filter_type\":2,\"lookup_filter_error_msg\":\"<target-name> không tồn tại hoặc không khớp với tiêu chí lọc.\",\"lookup_filter_status\":false,\"related_list_filter_related_list_id\":null,\"related_list_filter_status\":false,\"allow_searching\":0}"
}
```

Stored field:

```json
{
  "fieldType": "reference",
  "fieldMetaData": "{\"related_list_filter_related_list_id\":null,\"link_field\":\"id\",\"lookup_filter\":\"FIs0P85HaezDj\",\"lookup_filter_status\":false,\"object_slug\":\"object_1\",\"lookup_filter_type\":2,\"lookup_filter_error_msg\":\"Object 1 không tồn tại hoặc không khớp với tiêu chí lọc.\",\"related_list_filter_status\":false,\"allow_searching\":0,\"object\":\"OT2THCQE8KMS4\"}"
}
```

⚠️ **Phát hiện**: BE convert `lookup_filter` từ object form thành **string id** `FIs0P85HaezDj` (filter id) khi lưu — saving inline filter as a separate filter resource.

### RULE-FIELD-DEPENDENCYLOOKUP-06: Khác biệt với `lookup_normal`

| Khía cạnh | `lookup_normal` | `reference` (dependency) |
|---|---|---|
| `fieldType` | `"lookup_normal"` | `"reference"` |
| Quan hệ | Loose reference | **Master-detail** ràng buộc |
| `required` | Tuỳ chọn | **Luôn `1`** (forced) |
| `multiple` | 0 hoặc 1 | **Luôn `0`** (single parent) |
| `meta_data.link_field` | `""` mặc định, hoặc slug field hiển thị | **Luôn `"id"`** |
| Đăng ký dependency BE | KHÔNG | **CÓ** — vào `linkObjects` của `get-dependants` |
| Xoá parent record | Child không bị xoá | Child **bị cascade delete** (verify) |
| `rollup_summary` từ parent | KHÔNG hỗ trợ | **HỖ TRỢ** — `get-dependants` trả child này |
| URL form | `/data-field/create/lookup` | `/data-field/create/lookup?lookup_type=dependency` |

### RULE-FIELD-DEPENDENCYLOOKUP-07: `get-dependants` endpoint

Để check object có child dependency nào, gọi:

```
GET /api/v1/objects/object/get-dependants/{parentObjectId}
```

Response:

```json
{
  "data": {
    "mainObjects": [],
    "childrenObjects": [],
    "linkObjects": [
      {
        "objectId": "<child-id>",
        "fieldId": "<child-dep-field-id>",
        "objectName": "<child-name>",
        "fieldName": "<dep-field-name>",
        "objectSlug": "<child-slug>",
        "fieldSlug": "<dep-field-slug>"
      }
    ]
  }
}
```

`linkObjects` chứa tất cả child có `fieldType: "reference"` + `link_field: "id"` trỏ về parent này. `mainObjects` / `childrenObjects` rỗng trong workspace test thực tế — có thể dành cho schema khác (verify thêm khi gặp).

### RULE-FIELD-DEPENDENCYLOOKUP-08: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`, `object`, `link_field`, `related_list_slug`. Có thể sửa: `name`, `description`, `lookup_filter` (advanced filter). Xoá field dependency → cascade xoá related list trên target + **tất cả record con liên quan**.
