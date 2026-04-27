---
title: Kiến thức về Related List
impact: HIGH
impactDescription: Filter sai field path (slug vs slug.id) cho lookup_normal/reference dẫn đến không lấy được bản ghi related list; related list mặc định status=0 (inactive)
tags: related-list, lookup, reference, filter, ui-kit
---

## Kiến thức về Related List

### Khái niệm

Khi một field `lookup_normal` hoặc `reference` (dependency) — gọi chung là **trường lookup** — được tạo trong **Object A** trỏ sang **Object B** → ở **Object B tự động tạo ra một related list** tương ứng. KHÔNG có API tạo related list riêng — BE tự sinh ngay khi field lookup được tạo.

### RULE-RL-01: Cấu trúc Related List

Schema đầy đủ trả về từ `withRelatedList=1`:

| Key | Mô tả |
|-----|-------|
| `id` | ID của related list (vd `RL0UCCQEJKM92`) |
| `name` | Tên hiển thị (lấy từ `related_list_name` lúc tạo field) |
| `slug` | Slug định danh (lấy từ `related_list_slug` lúc tạo field) |
| `lookupType` | `"lookup_normal"` / `"reference"` |
| `status` | `0` mặc định (**inactive** — không hiện trên record); `1` khi user kích hoạt |
| `sort` | Thứ tự hiển thị trong tab "Liên quan" |
| `displayColumn` | `null` mặc định — JSON config column nào hiện trên record (set qua UI Cài đặt) |
| `minRecord` / `maxRecord` | `null` mặc định — ràng buộc số record liên quan |
| `sourceObjectId` / `sourceObjectSlug` | Object A (chứa trường lookup) |
| `sourceFieldId` / `sourceFieldSlug` | Trường lookup trong Object A |
| `lookupObjectId` / `lookupObjectSlug` | Object B (target — đối tượng "chủ" của related list) |
| `sourceFieldMetaData` | JSON string — mirror `meta_data` của field lookup |
| `sourceFieldIsMultiple` | `0`/`1` — lấy từ `multiple` của field lookup (quyết định op filter, xem RULE-RL-05) |

### RULE-RL-02: Lấy danh sách Related List của Object

Gọi API chi tiết object kèm tham số `withRelatedList = 1`:

```typescript
const { object } = useGetDataFieldBySlug({
  objectSlug: "objectB_slug",
  // ... các params khác
});
// Truyền withRelatedList = 1 trong API call để nhận danh sách related list
```

### RULE-RL-03: Lấy bản ghi của Related List — 3 bước

**Bước 1**: Lấy chi tiết related list từ object B (với `withRelatedList = 1`).

**Bước 2**: Lấy chi tiết field lookup theo `sourceObjectSlug` + `sourceFieldSlug`.

**Bước 3**: Gọi `useFilterRecord` / `useInfinityRecord` với `object_slug = sourceObjectSlug` và filter phù hợp.

### RULE-RL-04: Filter `field` phụ thuộc field type lookup

Filter dùng cấu trúc `FilterGeneratorItem`:

```typescript
interface FilterGeneratorItem {
  field: string | null;       // slug field cần filter
  op: Operator | null;        // "=", "in", "!=", "is null", ...
  params: FilterParamsType;   // giá trị filter
}
```

| Field type | `field` của filter |
|---|---|
| `lookup_normal` | `sourceFieldSlug` |
| `reference` | `<sourceFieldSlug>.id` |

### RULE-RL-05: Operator phụ thuộc `multiple` của field lookup

```typescript
// Object A "Đơn hàng" có field "customer" (reference) trỏ sang Object B "Khách hàng"
// → Object B có related list với sourceFieldSlug = "customer"
// → Lấy bản ghi của khách hàng id = "customer-123"

const isLookupNormal = lookupField?.fieldType === "lookup_normal";
const recordBId = "customer-123";

// Single value → op "="
const filters: FilterGeneratorItem[] = [{
  field: isLookupNormal ? "customer" : "customer.id",
  op: "=",
  params: recordBId,
}];

// Multiple values → op "in"
const filtersMultiple: FilterGeneratorItem[] = [{
  field: isLookupNormal ? "customer" : "customer.id",
  op: "in",
  params: [recordBId],
}];

const { data } = useFilterRecord({
  object_slug: relatedList.sourceObjectSlug,
  filters,
});
```

### RULE-RL-06: Cách BE tự sinh Related List khi tạo field lookup

Tên + slug của related list nằm ở **top-level** của payload `POST /api/v1/objects/field/create`, KHÔNG nằm trong `meta_data`:

```json
{
  "type": "lookup_normal",
  "object_type_id": "<object-A-id>",
  "name": "Test User Lookup",
  "slug": "test_user_lookup",
  "related_list_name": "Test ShortText - User RelatedList",
  "related_list_slug": "test_shorttext_user_rl",
  "meta_data": "{\"object\":\"<object-B-id>\",\"object_slug\":\"<object-B-slug>\",\"multiple_limit\":{\"min\":0,\"max\":30},\"link_field\":\"\"}"
}
```

Sau khi tạo, query `GET object B with withRelatedList=1` thấy related list mới với `sourceFieldSlug = <field-slug-vừa-tạo>`, `lookupObjectSlug = B.slug`, `status = 0`.

### RULE-RL-07: `status = 0` mặc định — Related list ẩn cho đến khi kích hoạt

Mới sinh, related list ở trạng thái **inactive** (`status: 0`) — KHÔNG hiển thị block "Liên quan" trên record của Object B. Để hiện cần vào `Settings → Object B → Danh sách liên quan → <related-list> → Cài đặt` và bật. BE patch `status = 1`.

⚠️ Khi code feature liên quan đến related list, đừng giả định mọi related list từ `withRelatedList=1` đều active — phải lọc `status === 1` nếu chỉ muốn hiển thị bản đã kích hoạt.

Ngoại lệ: Related list của 2 standard field `created_by` / `updated_by` cũng có `status = 0` mặc định (auto-tạo bởi `AUTOMATION-CGV` — xem ví dụ `Created Test ShortText Obj`, `Updated Test ShortText Obj` với `sort = 151`).

### RULE-RL-08: UI quản lý — `/settings/object/{slug}/related-list`

Trang này list mọi related list trên object đó, mỗi card hiển thị:
- Tên related list + link ngược về `sourceObject`
- Danh sách field của `sourceObject` (gồm cả custom + standard) để chọn `displayColumn` qua dialog "Cài đặt"
- Drag handle để đổi `sort`
- Nút bật/tắt `status`

`displayColumn` lưu JSON các slug field nguồn được hiển thị khi user expand related list trên record của Object B (vd `["name", "test_short_text_field"]`). `null` = dùng default (`name`).

### RULE-RL-09: Khi nào related list bị xoá

| Action | Hệ quả |
|---|---|
| Xoá field lookup (`lookup_normal`/`reference`) | BE cascade **xoá related list** trên target |
| Xoá field `reference` (dependency) | Cascade xoá related list trên target + **xoá toàn bộ record con** |
| Xoá object A (chứa field lookup) | Cascade xoá field + related list trên B |
| Xoá object B (target) | Field lookup trên A invalid (BE set null hoặc xoá tuỳ config) |
