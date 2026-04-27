---
title: Kiến thức về Related List
impact: HIGH
impactDescription: Filter sai field path (slug vs slug.id) cho lookup_normal/reference dẫn đến không lấy được bản ghi related list
tags: related-list, lookup, reference, filter, ui-kit
---

## Kiến thức về Related List

### Khái niệm

Khi một field `reference` hoặc `lookup_normal` (gọi chung là **trường lookup**) được tạo trong **Object A** trỏ sang **Object B** → ở **Object B tự động tạo ra một related list** tương ứng.

### RULE-RL-01: Cấu trúc Related List

| Key | Mô tả |
|-----|-------|
| `id` | ID của related list |
| `name` | Tên hiển thị |
| `slug` | Slug định danh |
| `sourceObjectSlug` | Slug của **Object A** (chứa trường lookup) |
| `sourceFieldSlug` | Slug của **trường lookup** trong Object A |

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
