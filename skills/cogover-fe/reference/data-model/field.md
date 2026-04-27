---
title: Kiến thức về Field (DataField)
impact: HIGH
impactDescription: Field định nghĩa cấu trúc dữ liệu — chọn sai field type/metadata gây lỗi validation và rendering
tags: field, data-field, field-type, metadata, ui-kit
---

## Kiến thức về Field (DataField)

### Khái niệm

**Field (DataField)** định nghĩa cấu trúc của một trường dữ liệu trong object. Mỗi field có `slug`, `name`, `fieldType`, `fieldMetaData`, và các thuộc tính phân quyền/hiển thị.

### RULE-FIELD-01: Interface DataField

Các thuộc tính quan trọng:

| Property | Kiểu | Mô tả |
|---|---|---|
| `slug` | `string` | Định danh duy nhất, dùng làm key trong record |
| `name` | `string` | Tên hiển thị |
| `fieldType` | `DataTypes` | Loại field (xem RULE-FIELD-02) |
| `fieldMetaData` | `DataFieldMetaData` | Cấu hình chi tiết (format, limit, validation) |
| `required` | `number` | Bắt buộc (1) hay không (0) |
| `multiple` | `number` | Cho phép nhiều giá trị (1) hay không (0) |
| `options?` | `DataFieldOption[]` | Danh sách tùy chọn (cho `single_choice`, `multi_choices`, `radio_button`...) |
| `viewable`, `editable`, `creatable` | — | Quyền xem/sửa/tạo |
| `readOnly?` | `boolean \| number` | Chỉ đọc |
| `isStandard` | `number` | Field hệ thống (1) hay tùy chỉnh (0) |

### RULE-FIELD-02: 22 kiểu dữ liệu user-facing (Form Tạo trường)

Trên form `Settings → Object → {object} → Cấu hình trường → Tạo trường` user chọn 1 trong **22 kiểu dữ liệu** sau (đây là UI grouping, không phải `DataTypes` enum kỹ thuật):

| # | Tên hiển thị | Mô tả ngắn | DataTypes nội bộ |
|---|---|---|---|
| 1 | Văn bản ngắn (Short-text) | Văn bản 1 dòng, thông tin ngắn gọn | `short_text` |
| 2 | Số điện thoại (Phone) | Lưu số thuê bao điện thoại | `phone` |
| 3 | Boolean | Logic 2 giá trị: true / false | `boolean` |
| 4 | Văn bản dài (Long-text) | Văn bản nhiều dòng (paragraph) | `long_text` |
| 5 | Email | Địa chỉ email hợp lệ | `email` |
| 6 | Lựa chọn đơn (Single-choice / Radio button) | Chọn 1 giá trị từ danh sách xác định | `single_choice` (UI có thể render `radio_button`) |
| 7 | Thời gian (Datetime) | Ngày, Giờ, Thời lượng theo định dạng | `date`, `time`, `date_time`, `time_duration`, `date_range`, `time_range`, `date_time_range` |
| 8 | Đường dẫn liên kết URL | Lưu URL truy cập được | `url` |
| 9 | Lựa chọn nhiều (Multi-choices / Checkboxes) | Chọn nhiều giá trị từ danh sách | `multi_choices` (UI có thể render `checkbox`) |
| 10 | Số (Number) | Số nguyên hoặc thập phân, có min/max/precision | `numeric`, `decimal` |
| 11 | Nhãn (Label) | Phân loại, gán nhãn thông tin | `label` |
| 12 | Cây thư mục (Cascading) | Cấu trúc cây / phân nhánh nhiều cấp | `cascading` |
| 13 | Phần trăm (Percent) | Tỷ lệ %, ký hiệu `%` | `percent` |
| 14 | Tải lên tệp tin (File Upload) | Upload file (tài liệu, ảnh, audio, video) | `file` |
| 15 | Xếp hạng (Rating) | Thang điểm 1-5 (hoặc tùy chỉnh) | `rating` |
| 16 | Tiền tệ (Currency) | Giá trị tiền tệ có ký hiệu | `currency` |
| 17 | Biểu thức chính quy (Regex) | Validate input bằng pattern regex | `regex` |
| 18 | Tra cứu thường (Normal lookup) | Lookup tới object khác (single/multiple) | `lookup_normal`, `reference` |
| 19 | Đánh số tự động (Auto-number) | Tự sinh số định danh theo prefix/format/start/step | `auto_number` |
| 20 | Công thức (Formula) | Tính toán từ field khác, **read-only** | `formula` |
| 21 | Tra cứu phụ thuộc (Dependency lookup) | Quan hệ master-detail, ràng buộc đối tượng chính | `lookup_parent` |
| 22 | Tổng hợp (Roll-up Summary) | Sum / Count / Max / Min trên bản ghi liên quan qua dependency lookup | `rollup_summary` |

**Lưu ý**:
- 22 là số entry trong combobox **chọn kiểu dữ liệu** ở Bước 1 — không phải số phần tử trong `DataTypes` enum.
- Một entry UI có thể mapping ra **nhiều** `DataTypes` nội bộ tuỳ subtype (vd: "Thời gian" → 7 enum giá trị; "Số" → 2 giá trị; "Single-choice" có thể là `single_choice` hoặc `radio_button` tuỳ display mode).
- Một số `DataTypes` nội bộ không xuất hiện trong combobox tạo mới: `lookup_peer2peer`, `embedded`, `children`, `link`, `select_list` — chúng là **field hệ thống** hoặc do lookup ngược tự sinh, không tạo thủ công từ form này.

### RULE-FIELD-03: Các loại Field Type (DataTypes enum)

Định nghĩa tại `ui-kit/src/apis/dataField/dataField.type.ts` — đây là enum kỹ thuật API trả về.

| Nhóm | Field Types |
|------|-------------|
| Text | `short_text`, `long_text`, `label` |
| Lựa chọn | `single_choice`, `multi_choices`, `checkbox`, `boolean`, `radio_button`, `select_list` |
| Số | `numeric`, `decimal`, `percent`, `currency`, `auto_number` |
| Ngày giờ | `date`, `date_range`, `time`, `time_range`, `time_duration`, `date_time`, `date_time_range` |
| Liên kết | `lookup_parent`, `lookup_peer2peer`, `lookup_normal`, `embedded`, `reference`, `children`, `link` |
| Khác | `url`, `email`, `phone`, `file`, `regex`, `cascading`, `rating`, `formula`, `rollup_summary` |

### RULE-FIELD-04: fieldMetaData theo từng loại field

Mỗi field type có metadata riêng (vd: `ShortTextMetaData`, `NumberMetaData`, `DateMetaData`...) lưu trong `fieldMetaData`. Truy cập đúng kiểu metadata trước khi render UI hoặc validate.

### RULE-FIELD-05: Lấy thông tin field bằng useGetDataFieldBySlug

```typescript
import useGetDataFieldBySlug from "@stringeecom/ui-kit/hooks/useGetDataFieldBySlug";

const { fields, isLoading, object } = useGetDataFieldBySlug({
  objectSlug: "customer",
  dataFieldSlugs: ["name", "email", "phone"],
});

// fields: (DataField | null)[] — tương ứng thứ tự dataFieldSlugs truyền vào
```

Có thể dùng `dataFieldIds` thay cho `dataFieldSlugs`, và `objectId` thay cho `objectSlug`.

### RULE-FIELD-06: Field Lookup tự sinh Related List

Khi tạo field `reference` hoặc `lookup_normal` trong object A trỏ sang object B → object B tự sinh related list tương ứng. Xem reference `related-list`.

### RULE-FIELD-07: 6 trường mặc định khi tạo Object mới

Khi tạo 1 object mới, hệ thống **tự sinh 6 field tiêu chuẩn** (`isStandard = 1`, `creator = "Automation Cogover"`). Không thể xoá từ UI list (checkbox disabled).

| # | Tên hiển thị | Slug | DataType | Ghi chú |
|---|---|---|---|---|
| 1 | ID | `id` | `short_text` | Định danh duy nhất của bản ghi |
| 2 | Name | `name` | Phụ thuộc `name_field.type` của object: `short_text` **hoặc** `auto_number` | Trường tên bản ghi — chọn type lúc tạo object, **không sửa được sau** (xem `data-model/object` RULE-OBJECT-09) |
| 3 | Tạo lúc | `created` | `date_time` | Audit thời gian tạo |
| 4 | Cập nhật lúc | `updated` | `date_time` | Audit thời gian cập nhật cuối |
| 5 | Tạo bởi | `created_by` | `lookup_normal` → User | Audit người tạo |
| 6 | Cập nhật bởi | `updated_by` | `lookup_normal` → User | Audit người cập nhật cuối |

**Phân nhóm**:
- **Định danh**: `id`, `name`
- **Audit thời gian**: `created`, `updated`
- **Audit người dùng**: `created_by`, `updated_by`

**Lưu ý quan trọng**:
- 6 field này **không** xuất hiện trong combobox "Tạo trường" (RULE-FIELD-02) vì là field hệ thống.
- `created_by`/`updated_by` là `lookup_normal` → 2 related list này tự sinh trên object `User` (xem `data-model/related-list`).
- Khi gọi API list field của object mới (chưa tạo field tuỳ chỉnh nào) sẽ luôn trả về tối thiểu 6 entry này.
- Tham chiếu chéo: trùng nội dung với **RULE-OBJECT-10** trong `data-model/object` — giữ ở 2 nơi vì cùng là kiến thức nền tảng cho cả Object và Field.
