## Cách sử dụng component FilePreview trong ui-kit

Đường dẫn: `ui-kit/src/components/FilePreview`

Hai variant:
- `FilePreview` — dùng với `UploadFile | ClientUploadResponse`, tự lấy URL public từ API.
- `BaseFilePreview` — dùng với `FilePreviewItem[]`, hỗ trợ `getPublicUrl` tùy chỉnh và cache URL.

### Props chung

| Prop | Kiểu | Mặc định | Mô tả |
|------|------|----------|-------|
| `files` | `FilePreviewItem[]` hoặc `(UploadFile \| ClientUploadResponse)[]` | bắt buộc | Danh sách file |
| `activeIndex` | `number` | `0` (nội bộ) | Chỉ số file đang hiển thị (controlled) |
| `onChangeActiveIndex` | `(index: number) => void` | — | Callback khi chuyển file |
| `scaleStep` | `number` | `0.5` | Bước zoom |
| `minScale` | `number` | `1` | Zoom nhỏ nhất |
| `maxScale` | `number` | `10` | Zoom lớn nhất |
| `onClose` | `() => void` | — | Callback khi đóng |

### FilePreviewItem (BaseFilePreview)

| Prop | Kiểu | Mô tả |
|------|------|-------|
| `id` | `string` | ID file |
| `url` | `string` | URL trực tiếp |
| `ext` | `string` | Phần mở rộng (`"pdf"`, `"png"`) |
| `fileName` | `string` | Tên hiển thị |
| `getPublicUrl` | `() => Promise<string>` | Lấy URL public (tùy chọn, kết quả cache) |

---

### RULE-FP-01: Chọn đúng variant

**Sai:**
```tsx
// ❌ Tự tạo object rồi truyền vào FilePreview
<FilePreview files={[{ id: "1", url: "...", ext: "png", fileName: "f.png" }]} onClose={onClose} />
```

**Đúng:**
```tsx
// ✅ Dữ liệu từ upload — dùng FilePreview
<FilePreview files={uploadedFiles} onClose={onClose} />

// ✅ Dữ liệu đã chuẩn hóa — dùng BaseFilePreview
<BaseFilePreview files={[{ id: "1", url: "...", ext: "png", fileName: "f.png" }]} onClose={onClose} />
```

---

### RULE-FP-02: Render có điều kiện để mở/đóng — không có prop `open`

```tsx
// ❌ Không có prop open
<FilePreview open={isOpen} files={files} onClose={onClose} />

// ✅ Dùng conditional rendering
{isOpen && <FilePreview files={files} onClose={onClose} />}
```

---

### RULE-FP-03: Truyền mảng `files` ít nhất một phần tử

```tsx
// ❌ Mảng rỗng gây lỗi runtime
<FilePreview files={[]} onClose={onClose} />

// ✅ Kiểm tra trước khi render
{files.length > 0 && <FilePreview files={files} onClose={onClose} />}
```

---

### RULE-FP-04: Dùng `getPublicUrl` trong BaseFilePreview cho file cần xác thực

```tsx
// ✅ Truyền getPublicUrl để component tự xử lý và cache
<BaseFilePreview
  files={[{
    id: file.id, url: file.url, ext: "pdf", fileName: "doc.pdf",
    getPublicUrl: () => getSignedUrl(file.id),
  }]}
  onClose={onClose}
/>
```
