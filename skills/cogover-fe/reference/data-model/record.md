---
title: Kiến thức về Record
impact: HIGH
impactDescription: Record chứa dữ liệu thực tế — hiểu sai cấu trúc/quyền/format payload gây lỗi hiển thị, mất permission check, BE từ chối tạo/sửa record
tags: record, data, query, mutation, permission, payload, field-format, ui-kit
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

### RULE-RECORD-06: Payload format khi tạo/sửa record (`POST /api/v1/records`)

Body shape:

```json
{
  "workspace_id": "<ws-id>",
  "object_type": "<object-type-id>",
  "data": { /* field values theo slug */ },
  "client_time_zone": "Asia/Saigon"
}
```

Lưu ý: trong `data` thường lặp lại `object_type` + `workspace_id` ở cuối — đây là duplicate FE gửi kèm, BE chấp nhận.

Mapping `fieldType` → JSON value gửi lên (verify từ payload thực tế trên `long02.release.cogover.net`, object `test`):

| Field type | Đơn trị | Đa trị (`multiple = 1`) | Ghi chú |
|---|---|---|---|
| `short_text`, `name` | `"Short text"` | `["Short text 1","Short text 2"]` | string thuần |
| `phone` | `"+85123213333"` | `["+12313123"]` | string, **dính liền prefix + số** (UI hiện "+85 123…", payload không space) |
| `boolean` | `true` / `false` | — | KHÔNG có biến thể multi |
| `long_text` | `{"value":"Long text","text_type":1}` | — | object — `text_type:1` = plain text |
| `email` | `"email@gmail.com"` | `["email@gmail.com"]` | |
| `single_choice` | `"option_1"` | — | **slug** của option, không phải id/label |
| `multi_choices` | — | `["option_1","option_2"]` | array slug |
| `date` | `"2026-04-15"` | `["2026-04-16","2026-04-16"]` | ISO `YYYY-MM-DD` **string** |
| `date_range` | `{"start":"2026-04-14","end":"2026-04-17"}` | — | object 2 ISO date |
| `date_time` | `1776145500000` | `[1777268700000]` | epoch **milliseconds integer** — KHÁC `date` |
| `date_time_range` | `{"start":1777268700000,"end":1777269600000}` | — | object 2 epoch ms |
| `time` | `44100000` | `[45000000]` | **ms-of-day** từ 00:00 (vd `12:15 PM` = 12.25 × 3600000 = 44.100.000) |
| `time_range` | `{"start":45900000,"end":47700000}` | — | object 2 ms-of-day |
| `time_duration` | `10000000000` | `[1000000, 2000000000]` | tổng số **ms** của khoảng thời gian |
| `url` | `{"url":"https://google.com","alias":""}` | `[{"url":"...","alias":""}]` | object — `alias` là label hiển thị |
| `numeric` | `12` | `[12]` | integer thuần |
| `decimal` | `12.2` | `[12.4]` | float |
| `label` | `"1"` | `["1","2","3"]` | string single, array khi multi |
| `cascading` | `["Node_1_1"]` | `["Node_1_1","Node_2_1"]` hoặc `["Node_1","Node_1_1","Node_2","Node_2_1"]` | LUÔN là array slug (kể cả single = array 1 leaf). Multi nhiều nhánh → flatten cả parent + leaf vào cùng array |
| `percent` | `0.75` (75%), `0.01` (1%), `null` (rỗng) | `[0.01]` | **ratio** — UI nhập `%` thì payload chia 100 |
| `file` | object metadata (xem dưới) | array các object | Phải upload trước qua `POST /api/v1/file/upload/v2/client_upload` để lấy `file_id` |
| `rating` (sao) | `3` | — | integer = số sao filled |
| `rating` (reaction) | `3` | — | integer = index reaction (0-based: angry=0, sad=1, confused=2, like=3, love=4) |
| `currency` | `11111` | `[111]` | **integer/float thuần**, không format `.` `,` (UI hiển thị `"11.111 VND"` chỉ là display) |
| `regex` | `"111"` | `["222"]` | string đã match pattern |
| `lookup_normal` | `"OBJOJL0QVEKNTF"` | `["OBJOJL0QVEKNTF"]` | record id |
| `reference` (dependency) | `"OBJOJL0QVEKNTF"` | — | record id, **luôn single** (master-detail) |

Field rỗng → gửi `null` (không gửi `""` hay `0`). Ví dụ `percent: null, percent_integer: null`.

Object `file`:

```json
{
  "fileName": "Chat---Cogover.pdf",
  "fileSize": 9205,
  "file_id": "asia-1_3E_FANVQEKRYB",
  "resolutionSizes": [],
  "acl": "private",
  "fileExt": "pdf",
  "fileServerUrlTemplate": "//asia-1{FILE_SERVER_SUFFIX_PUBLIC_URL}/{SHORT_LIVED_TOKEN}/asia-1_3E_FANVQEKRYB/Chat---Cogover.pdf",
  "fileType": "object_field",
  "url": "https://long02.release.cogover.net/files/{file_id}/original/Chat---Cogover.pdf?redirect=true"
}
```

Response thành công: `{"r":0,"msg":"OK","data":{"id":"<record-id>"}}`.

### RULE-RECORD-07: Pitfalls dễ sai khi build form record

1. **Date vs Date time khác hoàn toàn**: `date` = ISO string `"2026-04-15"`, `date_time` = epoch ms integer `1776145500000`. KHÔNG nhầm lẫn.
2. **Time là ms-of-day, không phải epoch**: chỉ là ms từ `00:00` cùng ngày, không có tham chiếu đến ngày cụ thể.
3. **Percent là ratio**: user nhập `1` (1%) → payload `0.01`. Khi đọc record về để hiển thị phải nhân 100.
4. **Currency raw integer**: dù UI format `"11.111 VND"`, raw payload là số thuần (`11111`). Khi parse value cũ về form, dùng integer rồi để input field tự format.
5. **Phone không có space**: dù UI hiển thị `"+85 123213333"`, payload là `"+85123213333"`.
6. **Cascading luôn array**: kể cả single value → array 1 leaf slug. Multi mà chọn nhiều nhánh → array phẳng cả parent + leaf cùng cấp.
7. **Lookup vs Reference id**: cả 2 đều gửi record id string thuần (không phải `{id:"..."}`); khác biệt: `lookup_normal` có thể là array khi multi, `reference` luôn single.
8. **URL/Long text là object, không phải string**: dễ sai khi map từ form FormData → payload nếu wire input type="text" trực tiếp.
9. **Single/Multi choice gửi slug**: option `{id, slug, label}` — payload chỉ chứa `slug`, không gửi id hay label.
10. **`client_time_zone` luôn gửi**: BE dùng để diễn giải `date`/`date_time` đúng múi giờ user.

### RULE-RECORD-08: `_attachments` — đăng ký file của rich text để file không bị xoá

**Mọi object có field `long_text` (rich text) BẮT BUỘC tồn tại field hệ thống `_attachments`** (slug `_attachments`). Đây là field kho lưu metadata cho **mọi file/image user upload qua rich text editor**.

**Tại sao cần?**

Khi user upload file qua rich text (vd. nút "Image" của CKEditor), FE gọi `POST /api/v1/file/upload/v2/client_upload` → BE trả về `file_id` + URL. File lúc này là **orphan** (chưa thuộc record nào) — sau TTL nếu không được đăng ký với 1 record, BE sẽ **tự xoá** khiến `<img>` trong rich text bị broken.

`_attachments` là cơ chế để FE **đăng ký** các file này thuộc record nào → BE mark `file_id` ↔ `record_id` → file tồn tại vĩnh viễn cùng record.

**FE phải làm 2 việc trong cùng 1 POST `/api/v1/records`:**

1. **Insert `<img>` vào `content.value`** với src = URL resolved + query `?redirect=true&long_text_field_id=<rich-text-field-id>`. KHÔNG nhúng base64 metadata vào URL.
2. **Push metadata file vào `_attachments` array** (top-level trong `data`):

```json
"_attachments": [
  {
    "fileName": "image.png",
    "fileSize": 12994,
    "resizable": true,
    "file_id": "asia-1_3J_RZ8KQEKM72",
    "resolutionSizes": ["original","tiny","small","medium","large"],
    "acl": "private",
    "fileExt": "png",
    "fileServerUrlTemplate": "//asia-1{FILE_SERVER_SUFFIX_PUBLIC_URL}/{SHORT_LIVED_TOKEN}/asia-1_3J_RZ8KQEKM72/image.png",
    "fileType": "object_field",
    "url": "https://long04.cogover.net/files/{file_id}/{resolution_size}/image.png?redirect=true&long_text_field_id=OF0FDEQE8KCN4"
  }
]
```

⚠️ **Phân biệt URL ở 2 chỗ**:
- Trong `<img src>` của HTML: URL **resolved** (file_id và resolution thực: `asia-1_3J_RZ8KQEKM72/original/image.png`).
- Trong `_attachments[i].url`: URL **template** với placeholder `{file_id}` và `{resolution_size}` (URL-encoded thành `%7B...%7D`) — BE/FE substitute lúc render tuỳ resolution cần.

Cả 2 chỗ đều có query param `long_text_field_id=<rich-text-field-id>` để BE biết file thuộc field rich text nào.

**Hệ quả khi build feature record có rich text:**

- Object schema phải có `_attachments` (BE auto-tạo khi tạo field `long_text` đầu tiên — verify thêm).
- FE form rich text phải **collect tất cả file user upload** (qua CKEditor upload adapter) và build `_attachments` array khi submit.
- Khi update record (xoá ảnh khỏi rich text), FE phải **remove metadata tương ứng khỏi `_attachments`** để BE biết file đó không còn dùng → có thể GC sau.
- Nếu thiếu bước 2 (chỉ insert HTML, không push `_attachments`) → file orphan, image biến mất sau ít phút/giờ.
