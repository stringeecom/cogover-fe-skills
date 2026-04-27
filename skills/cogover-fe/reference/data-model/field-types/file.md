---
title: Field Type — Tải lên tệp tin (File Upload)
impact: HIGH
impactDescription: max_size lưu bytes, không phải MB; file_group_type chuyển 5 mode (file/media/file-media/avatar/all); is_public/is_resizable lưu boolean (type def là number)
tags: data-field, file, upload, UploadFileMetaData, ui-kit
---

## Field Type — Tải lên tệp tin (File Upload)

`fieldType = "file"` (`DataTypes`). UI label: **"Tải lên tệp tin (File Upload)"**.

URL form tạo: `/settings/object/{objectSlug}/data-field/create/upload-file`.

Cấu hình common: xem `field-types/common`.

### RULE-FIELD-FILE-01: Form fields riêng

| # | UI label | Form name | Default | Mô tả |
|---|---|---|---|---|
| 1 | Loại dữ liệu | `fileGroupType` | `"Tệp tin"` (`"file"`) | Combobox 5 option: file / media / file-media / avatar / all |
| 2 | Các định dạng cho phép tải lên | `fileType` | 15 mặc định: doc/docx/xlsx/xls/csv/ppt/pptx/pdf/txt/rtf/html/htm/zip/json/md | Multi-select |
| 3 | Công khai | `isPublic` | `false` | Checkbox — bật cho phép xem file không cần auth |
| 4 | Dung lượng cho phép tải lên (MB) — Tối đa | `maxSize` (UI nhập MB) | `50` | NumberInput — UI nhận MB, BE lưu bytes |
| 5 | Giá trị mặc định | (file picker) | (empty) | Drop zone upload file mặc định |

### RULE-FIELD-FILE-02: Input common bị ẩn

So với common (RULE-FIELD-COMMON-02):
- ❌ KHÔNG có Placeholder (`hint`) — file không có placeholder
- ❌ KHÔNG có "Cho phép sử dụng để Tìm kiếm nhanh" (`quickSearch`)
- ❌ KHÔNG có "Tính duy nhất" (`isUnique`)

### RULE-FIELD-FILE-03: Mapping form → `meta_data` (`UploadFileMetaData`)

Reference: `ui-kit/src/apis/dataField/dataField.type.ts:378-399`.

```typescript
export interface UploadFileMetaData {
    multiple_limit?: { min?: number; max?: number };
    max_size?: number;                   // bytes
    file_type?: string[];
    file_group_type?: "file" | "media" | "file-media" | "avatar" | "all";
    is_public?: number;                  // ⚠️ type def: number, BE thực tế: boolean
    is_resizable?: number;               // ⚠️ type def: number, BE thực tế: boolean
    autoGenerateAvatar?: boolean;
    autoGenerateAvatarFieldId?: string;
    is_signing_file?: boolean;
    signing_position?: (SigningPositionItem | string)[];
    preview_file?: boolean;
    edit_file?: boolean;
    max_width_file?: number;             // px, 0 = không giới hạn
    max_height_file?: number;
}
```

| Form key | Đường dẫn trong `meta_data` |
|---|---|
| `fileGroupType` | `file_group_type` |
| `fileType` (array extensions) | `file_type` |
| `isPublic` | `is_public` (boolean ở thực tế) |
| `maxSize` (UI MB) × 1024² | `max_size` (bytes) |
| `multipleLimit` | `multiple_limit.min/max` |

### RULE-FIELD-FILE-04: Payload thực tế (verify từ API)

Tạo "Test File Field" với defaults (file group, 15 extensions, max 50MB, không public):

```json
{
  "fieldType": "file",
  "fieldMetaData": "{\"multiple_limit\":{\"min\":0,\"max\":200},\"max_size\":52428800,\"file_type\":[\"doc\",\"docx\",\"xlsx\",\"xls\",\"csv\",\"ppt\",\"pptx\",\"pdf\",\"txt\",\"rtf\",\"html\",\"htm\",\"zip\",\"json\",\"md\"],\"file_group_type\":\"file\",\"is_public\":false,\"is_resizable\":false}"
}
```

⚠️ Phát hiện:
- `max_size = 52428800` bytes = **50 MB** (UI hiện 50, BE × 1024² lưu bytes)
- `is_public`, `is_resizable` là **boolean** (`false`/`true`), không phải number 0/1 như type def
- `multiple_limit.max = 200` (cao hơn các type khác — file cho upload nhiều)

### RULE-FIELD-FILE-05: 5 mode `file_group_type`

| Value | UI label | File extension cho phép |
|---|---|---|
| `"file"` | Tệp tin | doc, docx, xls, csv, pdf, ppt, txt... (text files) |
| `"media"` | Đa phương tiện | mp4, mp3, wav, avi, mov, jpg, png... (audio/video/image) |
| `"file-media"` | Tệp + Media | Cả 2 |
| `"avatar"` | Avatar | jpg, png, jpeg, webp (chỉ image cho profile pic) |
| `"all"` | Tất cả | Mọi extension (validate qua MIME thay vì ext) |

### RULE-FIELD-FILE-06: Render record

- Single file (`multiple = 0`): drop zone + 1 file uploaded preview
- Multiple files: drop zone + list files với preview thumbnail (image) / icon (doc)
- `is_public = true`: link download không cần auth (CDN URL công khai)
- `preview_file = true`: click file → modal preview thay vì download

### RULE-FIELD-FILE-07: Lưu ý khi sửa

Sau khi tạo, **không sửa được**: `slug`, `type`. Có thể sửa: `file_type`, `max_size`, `is_public`, `multiple_limit`. Đổi `file_group_type` từ `file` → `media` thường không cho vì validate khác → tạo field mới.
