---
title: Kiến thức về Record
impact: HIGH
impactDescription: Record chứa dữ liệu thực tế — hiểu sai cấu trúc/quyền gây lỗi hiển thị, mất permission check
tags: record, data, query, mutation, permission, ui-kit
---

## Kiến thức về Record

### Khái niệm

**Record** là một bản ghi cụ thể của object. Các key trong record chính là `slug` của field thuộc object đó.

### RULE-RECORD-01: Cấu trúc của RecordItem

- Key là `slug` của các field thuộc object
- Luôn có `id: string`
- Có thể có `name?: string`
- `_canNotViewColumns?: string[]` — slug field user không được **xem** (KHÔNG trả về key đó trong record)
- `_canNotUpdateColumns?: string[]` — slug field user không được **cập nhật**

```typescript
// Ví dụ record của object "Khách hàng"
{
  id: "abc-123",
  name: "Nguyễn Văn A",        // slug: name
  email: "a@example.com",       // slug: email
  phone: "0901234567",          // slug: phone
  // status không trả về vì nằm trong _canNotViewColumns
  _canNotViewColumns: ["status"],
  _canNotUpdateColumns: ["email"],
}
```

### RULE-RECORD-02: Hook query record

Tất cả nằm trong `ui-kit/src/apis/record/record.api.ts`:

| Hook | Mục đích | Params chính |
|------|----------|--------------|
| `useQueryRecordDetail` | Chi tiết 1 record | `{ id, objectId?, objectSlug? }` |
| `useFilterRecord` | Lọc danh sách record | `FilterRecordParams` (filters, sorts, size, search_after) |
| `useInfinityRecord` | Lọc record với infinite scroll | `FilterRecordParams` |
| `useGetRecordListByIds` | Lấy nhiều record theo IDs | `{ ids, objectId?, objectSlug? }` |
| `useLookupRecord` | Tìm record cho lookup field | `LookupRecordParams` |
| `useFilterRecordByGroup` | Lọc record theo nhóm | `FilterRecordByGroupParams` |
| `useGroupRecord` | Đếm record theo group | `GroupRecordParams` |
| `useGetRecordValueDetailByPath` | Lấy giá trị chi tiết theo path | `{ recordId, paths, objectId? }` |

### RULE-RECORD-03: Hook mutation record

| Hook | Mục đích | Params chính |
|------|----------|--------------|
| `useCreateRecord` | Tạo record mới | `CreateRecordParams { objectId, data }` |
| `useUpdateRecord` | Cập nhật record | `UpdateRecordParams { id, objectId, data }` |
| `useDeleteRecord` | Xóa record | `DeleteRecordParams { objectId, ids }` |
| `useBatchUpdateRecord` | Cập nhật nhiều record | `BatchUpdateRecordParams { objectId, data[] }` |
| `useSyncRecord` | Đồng bộ dữ liệu giữa các record | `SyncRecordParams` |
| `useMergeRecords` | Gộp nhiều record | `MergeRecordPayload` |

### RULE-RECORD-04: Kiểm tra quyền view/update trước khi hiển thị

Khi hiển thị hoặc cho phép chỉnh sửa field trong record, PHẢI kiểm tra `_canNotViewColumns` và `_canNotUpdateColumns`:

```typescript
const canView = !record._canNotViewColumns?.includes(fieldSlug);
const canUpdate = !record._canNotUpdateColumns?.includes(fieldSlug);
```

### RULE-RECORD-05: Hook hỗ trợ liên quan đến record

| Hook | Mục đích | Reference |
|------|----------|-----------|
| `useGetRecordPageLink` | Tạo link list/create/view record | `uikit-use-get-record-page-link` |
| `useMergeRecordFilters` | Merge nhiều bộ lọc record | `uikit-use-merge-record-filters` |
| `useReplaceRecordValue` | Thay variables `{$var.path}` bằng giá trị record | `uikit-use-replace-record-value` |
