---
title: Kiến thức về Object
impact: HIGH
impactDescription: Object là entity gốc — sai cấu trúc/cờ quyền/category dẫn đến hiển thị sai, mất permission, parse metaData lỗi
tags: object, data-model, schema, api, ui-kit
---

## Kiến thức về Object

### Khái niệm

**Object** đại diện cho một loại dữ liệu (ví dụ: "Khách hàng", "Đơn hàng", "Sản phẩm"). Mỗi object là một **entity gốc** trong mô hình dữ liệu, chứa:

- **Định nghĩa fields** — schema dữ liệu (xem reference `field`)
- **Related lists** — quan hệ ngược từ field lookup ở object khác (xem reference `related-list`)
- **Records** — bản ghi cụ thể thuộc object (xem reference `record`)

### RULE-OBJECT-01: Định danh bằng slug hoặc id

Object được xác định qua `objectSlug` hoặc `objectId`. Khi gọi API/hook, ưu tiên `objectSlug` vì code dễ đọc, dễ debug.

### RULE-OBJECT-02: Cấu trúc ObjectListResponse

Định nghĩa tại `ui-kit/src/apis/object/object.type.ts:62`. Mỗi item trong `data[]` chia làm các nhóm:

#### Định danh & nhãn

| Field | Type | Mô tả |
|---|---|---|
| `id` | `string` | ID object (vd: `"OTGVJ2QEEKRPB"`) |
| `slug` | `string` | Slug định danh duy nhất (vd: `"user"`) |
| `name` | `string` | Tên hiển thị (đã translate khi `translate=true`) |
| `originalName?` | `string` | Tên gốc, chưa translate |
| `pluralName` | `string` | Tên số nhiều |
| `workspaceId` | `string` | Workspace chứa object |

#### Phân loại & trạng thái

| Field | Type | Mô tả |
|---|---|---|
| `status` | `number` | 1 = active |
| `isStandard` | `number` | 1 = chuẩn hệ thống, 0 = tuỳ chỉnh |
| `type` | `number` | `ObjectTypeData`: `0=NORMAL`, `1=CHILDREN`, `2=JUNCTION` |
| `category` | `ObjectCategory` | `"normal" \| "workflow" \| "history"` |
| `typeInitId` | `number` | Phân loại khởi tạo |
| `description?` | `string` | Mô tả object |

#### Quyền cấp object

| Field | Type | Mô tả |
|---|---|---|
| `creatable` | `number` | Cho phép tạo record |
| `editable` | `number` | Cho phép sửa record |
| `viewable` | `number` | Cho phép xem record |
| `quickSearch` | `number` | Cho phép quick search |

> Quyền cấp **field** xem reference `field`. Quyền cấp **record** xem reference `record`.

#### Audit

| Field | Type | Mô tả |
|---|---|---|
| `created` / `updated` | `number` | Timestamp ms |
| `createdBy` / `updatedBy` | `string` | Personnel ID |
| `createdByInfo` / `updatedByInfo` | `ObjectAccountInfo` | Thông tin chi tiết người tạo/sửa (avatar, email, role, license, timezone, language, name…) |

#### Cấu hình & metadata

| Field | Type | Mô tả |
|---|---|---|
| `metaData` | `string` (JSON) | **Cần `JSON.parse`** ra `ObjectMetaData` (xem RULE-OBJECT-03) |
| `prefixId` | `string` | Prefix ID record |
| `elasticIndex` | `string` | Tên index ES |
| `uniqueKeys?` | `string` | Khóa unique |
| `parentFieldId?` | `string` | Field cha (cho `type=CHILDREN`) |
| `linkObjectType?` | `string` | Cho junction object |
| `name_field?` | `INameField` | Cấu hình trường tên (`name`, `type`, `meta_data`, `require`) |
| `duplicateRules` | `[]` | Rule chống trùng |
| `compositeKeys` | `any[]` | Composite key |

#### Vòng đời (xoá mềm)

| Field | Type | Mô tả |
|---|---|---|
| `remainDay` | `number` | Ngày còn lại trước khi xoá vĩnh viễn |
| `timeToDelete?` | `number` | Thời điểm sẽ xoá |

#### Relations (CHỈ có khi truyền tham số tương ứng)

| Field | Type | Khi nào có |
|---|---|---|
| `fields` | `DataField[]` | Khi `withFields=1`. Mặc định `[]` |
| `relatedLists` | `RelatedListItem[]` | Khi `withRelatedList=1`. Mặc định `[]` |

### RULE-OBJECT-03: Parse metaData trước khi dùng

`metaData` là **JSON string** — phải `JSON.parse` ra `ObjectMetaData`:

```typescript
interface ObjectMetaData {
  allow_direct_creation: number;
  is_config_field_name_with_view: number | boolean;
  config_field_name_with_view: "TEXT" | "HTML";
  config_text: string;
  config_text_formula: string;
  is_show_avatar_before: number | boolean;
  hide_create_button: boolean;
  enable_auto_fill?: number;
  enable_smart_paste?: number;
}

const meta: ObjectMetaData = JSON.parse(object.metaData);
```

### RULE-OBJECT-04: Lấy chi tiết Object kèm fields

Dùng hook `useGetDataFieldBySlug` để lấy thông tin object kèm danh sách field:

```typescript
import useGetDataFieldBySlug from "@stringeecom/ui-kit/hooks/useGetDataFieldBySlug";

const { fields, object, isLoading } = useGetDataFieldBySlug({
  objectSlug: "customer",
  dataFieldSlugs: ["name", "email", "phone"],
});

// object: ObjectListResponse — chi tiết object kèm tất cả fields
// fields: (DataField | null)[] — đúng thứ tự dataFieldSlugs truyền vào
```

Có thể dùng `objectId` thay cho `objectSlug`, và `dataFieldIds` thay cho `dataFieldSlugs`.

### RULE-OBJECT-05: Lấy Object kèm Related List

Truyền `withRelatedList = 1` khi cần load related list của object. Chi tiết tại reference `related-list`.

### RULE-OBJECT-06: Phân biệt Object Type

| `type` | Tên | Ý nghĩa |
|---|---|---|
| `0` | `NORMAL` | Object độc lập, đứng riêng |
| `1` | `CHILDREN` | Object con — có `parentFieldId` trỏ về object cha |
| `2` | `JUNCTION` | Object trung gian — nối 2 object qua nhiều-nhiều |

UI hiển thị "Tiêu chuẩn(Trung gian)" cho object có `isStandard=1` và `type=JUNCTION`.

### RULE-OBJECT-07: Phân biệt Object Category

| `category` | Ý nghĩa |
|---|---|
| `"normal"` | Object dữ liệu thường (Khách hàng, Đơn hàng…) |
| `"workflow"` | Object phục vụ quy trình (workflow instance, task…) |
| `"history"` | Object lưu lịch sử thay đổi |

Khi list object thường, filter `category = "normal"` (giống UI `/settings/object?category=normal`).

### RULE-OBJECT-08: API create object

**Endpoint**: `POST /api/v1/objects/object/create`

**Request body** (snake_case, **khác** với response camelCase):

```typescript
{
  name: string;                       // Tên hiển thị
  plural_name: string | null;         // Tên số nhiều, null nếu trống
  description: string;                // Mô tả, "" nếu trống
  slug: string;                       // Slug (immutable sau tạo)
  workspace_id: string;
  status: 0 | 1;                      // 1 = Đang hoạt động
  is_standard: 0 | 1;                 // Luôn 0 cho object tuỳ chỉnh
  type: 0 | 2;                        // 0 = NORMAL, 2 = JUNCTION
  meta_data: string;                  // JSON.stringify(ObjectMetaData)
  name_field: NameFieldPayload;       // xem RULE-OBJECT-09
  id: "";                             // luôn rỗng khi tạo, server tự sinh
  composite_keys: any[];              // [] nếu chưa cấu hình
}
```

**`meta_data` mặc định** khi tạo (đã `JSON.stringify`):
```json
{"is_config_field_name_with_view":0,"config_field_name_with_view":"TEXT","config_text":"","config_text_formula":"","is_show_avatar_before":0,"allow_report":0}
```

**Response thành công**:

```typescript
{
  r: 0,
  msg: "OK",
  data: {
    id: string,                       // ID object server sinh (vd "OTTPOTQETKMSA")
    slug: string,                     // = slug đã gửi
    options: [],
    standard_fields: Array<{          // 6 field hệ thống tự sinh — xem RULE-OBJECT-10
      id: string,
      slug: string,
      options: [],
    }>
  }
}
```

**Response lỗi quota** (status 422):
```json
{ "r": 534, "msg": "Exceed object per workspace max limit: 10" }
```

### RULE-OBJECT-09: name_field theo type

`name_field` định nghĩa trường "Tên bản ghi" (slug API luôn là `name`). UI form tạo chỉ cho 2 type — mỗi type có cấu trúc `meta_data` khác nhau:

#### `name_field.type = "short_text"`

```typescript
{
  name: "Name",                       // Display name (editable)
  type: "short_text",
  meta_data: JSON.stringify({
    character_limit: { min: 1, max: 255 },
    multiple_limit: { min: 1, max: 1 },
  }),
  require: 1,
}
```

Form chỉ hỏi `name` và `type` — `meta_data` là default cố định.

#### `name_field.type = "auto_number"`

```typescript
{
  name: "Name",
  type: "auto_number",
  meta_data: JSON.stringify({
    prefix: string,                   // Tiền tố, default ""
    suffix: string,                   // Hậu tố, default ""
    format: string,                   // Định dạng số, default "0000" (4 chữ số padding)
    start: number,                    // Số bắt đầu, default 1
    step: number,                     // Bước nhảy, default 1
    is_identifier: boolean,           // "Sử dụng trường để định danh URL", default false
  }),
  require: 1,
}
```

Khi chọn `auto_number`, form **expand thêm 7 control** để cấu hình các thuộc tính trên (Tiền tố, Hậu tố, Định dạng, Số bắt đầu, Bước nhảy, Sử dụng làm identifier URL, và Xem trước hiển thị giá trị mẫu sinh từ format+start).

> Form chỉ cho 2 type này vì field "Tên bản ghi" là **identity field cốt lõi** — không phù hợp với date/number/lookup.

### RULE-OBJECT-10: Standard fields tự sinh khi tạo object

Khi tạo object thành công, server **tự sinh 6 field hệ thống** mặc định cho mọi object (không cần khai báo trong request):

| Slug | Mô tả |
|---|---|
| `id` | ID record |
| `name` | Tên record (= field cấu hình ở `name_field`) |
| `created` | Timestamp tạo |
| `updated` | Timestamp cập nhật cuối |
| `created_by` | Personnel ID người tạo |
| `updated_by` | Personnel ID người cập nhật cuối |

Các field này luôn xuất hiện trong response `data.standard_fields[]` của API create, và sau đó luôn có trong `fields[]` khi gọi `withFields=1` ở API list.

### RULE-OBJECT-11: Form Tạo vs Form Sửa — thuộc tính immutable

Một số thuộc tính **chỉ chốt được khi tạo**, không sửa được sau:

| Field | Form tạo | Form sửa |
|---|---|---|
| `slug` | ✅ Editable | ❌ Disabled |
| `name_field.name` | ✅ Editable | ❌ Disabled |
| `name_field.type` (short_text / auto_number) | ✅ Editable | ❌ Disabled |
| `type` (NORMAL / JUNCTION) | ✅ Editable | ❌ Disabled |

Một số chỉ cấu hình được **sau khi tạo** (form tạo không có):

- `metaData.is_config_field_name_with_view` + 3 sub-field (`config_field_name_with_view`, `config_text`, `is_show_avatar_before`) — cấu hình hiển thị thay thế tên record (cần biết slug field hiện có để chèn template)
- `compositeKeys[]` — cần các field đã tồn tại

Lý do: cấu hình hiển thị thay thế cần chèn placeholder `{$field_slug}` (xem reference `record` RULE-RECORD-05 / `useReplaceRecordValue`), mà field thì phải tạo sau khi đã có object.

### RULE-OBJECT-12: Mapping form ↔ payload (create)

```
Form field                                  → Request body
────────────────────────────────────────────────────────────
Tên                                          → name
Tên số nhiều                                 → plural_name (null nếu trống)
Slug của đối tượng                           → slug
Mô tả                                        → description ("" nếu trống)
Tên trường (Record Name)                     → name_field.name
Kiểu dữ liệu = Short-text                    → name_field.type = "short_text"
                                                name_field.meta_data = {character_limit, multiple_limit}
Kiểu dữ liệu = Auto-number                   → name_field.type = "auto_number"
                                                name_field.meta_data = {prefix, suffix, format, start, step, is_identifier}
  ├─ Tiền tố                                 →   prefix
  ├─ Hậu tố                                  →   suffix
  ├─ Định dạng số tự động                    →   format
  ├─ Số bắt đầu                              →   start
  ├─ Bước nhảy                               →   step
  └─ Sử dụng trường để định danh URL         →   is_identifier
Ẩn nút tạo bản ghi mặc định                 → meta_data.hide_create_button
Trạng thái Đang hoạt động                    → status (1/0)
Cho phép báo cáo                             → meta_data.allow_report
Loại đối tượng (Thường/Trung gian)           → type (0=NORMAL / 2=JUNCTION)
```
