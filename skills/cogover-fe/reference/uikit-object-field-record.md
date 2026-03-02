---
title: Kiến thức về Object, Field, Record
impact: HIGH
impactDescription: Hiểu sai mô hình dữ liệu Object-Field-Record dẫn đến truy vấn sai, hiển thị sai, và lỗi permission
tags: object, field, record, related-list, data-model, api, hook
---

## Kiến thức về Object, Field, Record

### Mô hình dữ liệu

- **Object**: Đại diện cho một loại dữ liệu (ví dụ: "Khách hàng", "Đơn hàng"). Mỗi object chứa danh sách các **field**.
- **Field (DataField)**: Định nghĩa cấu trúc của một trường dữ liệu trong object. Mỗi field có `slug`, `name`, `fieldType`, `fieldMetaData`, và các thuộc tính khác.
- **Record**: Một bản ghi cụ thể của object. Các key trong record chính là `slug` của field.

### RULE-OFR-01: Cấu trúc của Record

Một record (`RecordItem`) có cấu trúc:
- Key là `slug` của các field thuộc object đó
- Luôn có `id: string`
- Có thể có `name?: string`
- `_canNotViewColumns?: string[]` — danh sách slug của các field mà user không được phép **xem**. Field nào không được xem thì **sẽ không trả về key đó** trong record.
- `_canNotUpdateColumns?: string[]` — danh sách slug của các field mà user không được phép **cập nhật**.

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

### RULE-OFR-02: Các loại Field Type (DataTypes)

Tham khảo `DataTypes` trong `ui-kit/src/apis/dataField/dataField.type.ts`:

| Nhóm | Field Types |
|------|-------------|
| Text | `short_text`, `long_text`, `label` |
| Lựa chọn | `single_choice`, `multi_choices`, `checkbox`, `boolean`, `radio_button`, `select_list` |
| Số | `numeric`, `decimal`, `percent`, `currency`, `auto_number` |
| Ngày giờ | `date`, `date_range`, `time`, `time_range`, `time_duration`, `date_time`, `date_time_range` |
| Liên kết | `lookup_parent`, `lookup_peer2peer`, `lookup_normal`, `embedded`, `reference`, `children`, `link` |
| Khác | `url`, `email`, `phone`, `file`, `regex`, `cascading`, `rating`, `formula`, `rollup_summary` |

Mỗi field type có metadata riêng (ví dụ: `ShortTextMetaData`, `NumberMetaData`, `DateMetaData`...) lưu trong `fieldMetaData`.

### RULE-OFR-03: Interface DataField

Các thuộc tính quan trọng của `DataField`:
- `slug: string` — định danh duy nhất, dùng làm key trong record
- `name: string` — tên hiển thị
- `fieldType: DataTypes` — loại field
- `fieldMetaData: DataFieldMetaData` — cấu hình chi tiết (format, limit, validation...)
- `required: number` — bắt buộc (1) hay không (0)
- `multiple: number` — cho phép nhiều giá trị (1) hay không (0)
- `options?: DataFieldOption[]` — danh sách tùy chọn (cho `single_choice`, `multi_choices`, `radio_button`...)
- `viewable`, `editable`, `creatable` — quyền xem/sửa/tạo
- `readOnly?: boolean | number` — chỉ đọc
- `isStandard: number` — field hệ thống (1) hay tùy chỉnh (0)

### RULE-OFR-04: Lấy thông tin field của object

Dùng hook `useGetDataFieldBySlug` để lấy thông tin data field:

```typescript
import useGetDataFieldBySlug from "@stringeecom/ui-kit/hooks/useGetDataFieldBySlug";

const { fields, isLoading, object } = useGetDataFieldBySlug({
  objectSlug: "customer",
  dataFieldSlugs: ["name", "email", "phone"],
});

// fields: (DataField | null)[] — tương ứng với thứ tự dataFieldSlugs truyền vào
// object: ObjectListResponse — thông tin object kèm tất cả fields
```

Cũng có thể dùng `dataFieldIds` thay cho `dataFieldSlugs`, và `objectId` thay cho `objectSlug`.

### RULE-OFR-05: Các hook lấy record

Tất cả hook record đều nằm trong `ui-kit/src/apis/record/record.api.ts`:

| Hook | Mục đích | Params chính |
|------|----------|--------------|
| `useQueryRecordDetail` | Lấy chi tiết 1 record | `{ id, objectId?, objectSlug? }` |
| `useFilterRecord` | Lọc danh sách record | `FilterRecordParams` (filters, sorts, size, search_after) |
| `useInfinityRecord` | Lọc record với infinite scroll | `FilterRecordParams` |
| `useGetRecordListByIds` | Lấy nhiều record theo IDs | `{ ids, objectId?, objectSlug? }` |
| `useLookupRecord` | Tìm record cho lookup field | `LookupRecordParams` |
| `useFilterRecordByGroup` | Lọc record theo nhóm | `FilterRecordByGroupParams` |
| `useGroupRecord` | Đếm record theo group | `GroupRecordParams` |
| `useGetRecordValueDetailByPath` | Lấy giá trị chi tiết theo path | `{ recordId, paths, objectId? }` |

### RULE-OFR-06: Các hook mutation record

| Hook | Mục đích | Params chính |
|------|----------|--------------|
| `useCreateRecord` | Tạo record mới | `CreateRecordParams { objectId, data }` |
| `useUpdateRecord` | Cập nhật record | `UpdateRecordParams { id, objectId, data }` |
| `useDeleteRecord` | Xóa record | `DeleteRecordParams { objectId, ids }` |
| `useBatchUpdateRecord` | Cập nhật nhiều record | `BatchUpdateRecordParams { objectId, data[] }` |
| `useSyncRecord` | Đồng bộ dữ liệu giữa các record | `SyncRecordParams` |
| `useMergeRecords` | Gộp nhiều record | `MergeRecordPayload` |

### RULE-OFR-07: Related List

#### Khái niệm

Khi một trường `reference` hoặc `lookup_normal` (gọi chung là **trường lookup**) được tạo trong **object A**, thì ở **object B** (object được trường lookup đó tham chiếu đến) sẽ **tự động tạo ra một related list** tương ứng nằm ở object B.

#### Lấy danh sách Related List của một Object

Gọi API lấy chi tiết object B và truyền thêm tham số `withRelatedList = 1`:

```typescript
// Ví dụ: lấy chi tiết object B kèm related list
const { object } = useGetDataFieldBySlug({
  objectSlug: "objectB_slug",
  // ... các params khác
});
// Truyền thêm withRelatedList = 1 trong API call để nhận được danh sách related list
```

#### Cấu trúc Related List

Mỗi related list có các key quan trọng sau:

| Key | Mô tả |
|-----|--------|
| `id` | ID của related list |
| `name` | Tên hiển thị của related list |
| `slug` | Slug định danh của related list |
| `sourceObjectSlug` | Slug của **object A** (object chứa trường lookup) |
| `sourceFieldSlug` | Slug của **trường lookup** trong object A |

#### Lấy danh sách bản ghi của Related List

Thực hiện theo 3 bước:

**Bước 1**: Lấy chi tiết related list từ object B (với `withRelatedList = 1` như trên).

**Bước 2**: Lấy chi tiết field lookup dựa vào `sourceObjectSlug` và `sourceFieldSlug`.

**Bước 3**: Dùng API lấy danh sách bản ghi với filter phù hợp. Filter sử dụng cấu trúc `FilterGeneratorItem` (xem `FilterGenerator/type.ts`):

```typescript
interface FilterGeneratorItem {
  field: string | null;  // slug của field cần filter
  op: Operator | null;   // toán tử: "=", "in", "!=", "is null", ...
  params: FilterParamsType; // giá trị filter
}
```

**Cách xây dựng filter phụ thuộc vào loại field lookup**:

- **Nếu field lookup là `lookup_normal`**: `field` = `sourceFieldSlug`
- **Nếu field lookup là `reference`**: `field` = `<sourceFieldSlug>.id`

```typescript
// Ví dụ: Object A = "Đơn hàng" có trường lookup "customer" (reference) tham chiếu đến Object B = "Khách hàng"
// → Object B "Khách hàng" sẽ có related list với sourceFieldSlug = "customer"
// → Lấy bản ghi related list cho khách hàng có id = "customer-123"

const isLookupNormal = lookupField?.fieldType === "lookup_normal";
const recordBId = "customer-123";

// Filter lấy các bản ghi thuộc related list
const filters: FilterGeneratorItem[] = [
  {
    field: isLookupNormal ? "customer" : "customer.id",
    op: "=",
    params: recordBId,
  },
];

// Nếu field lookup cho phép nhiều giá trị (multiple), dùng operator "in":
const filtersMultiple: FilterGeneratorItem[] = [
  {
    field: isLookupNormal ? "customer" : "customer.id",
    op: "in",
    params: [recordBId],
  },
];

// Truyền filters vào useFilterRecord hoặc useInfinityRecord
const { data } = useFilterRecord({
  object_slug: relatedList.sourceObjectSlug, // "order"
  filters,
});
```

### RULE-OFR-08: Kiểm tra quyền xem/sửa field trước khi hiển thị

Khi hiển thị hoặc cho phép chỉnh sửa field trong record, phải kiểm tra `_canNotViewColumns` và `_canNotUpdateColumns`:

```typescript
// Kiểm tra quyền xem
const canView = !record._canNotViewColumns?.includes(fieldSlug);

// Kiểm tra quyền sửa
const canUpdate = !record._canNotUpdateColumns?.includes(fieldSlug);
```
