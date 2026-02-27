---
title: Cách sử dụng component FilePreview trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến việc chọn sai variant component, truyền sai kiểu dữ liệu files, và thiếu xử lý điều hướng file
tags: file-preview, media, image, video, audio, document, zoom, ui-kit
---

## Cách sử dụng component FilePreview trong ui-kit

Đường dẫn component: `ui-kit/src/components/FilePreview`

Component có hai variant:
- `FilePreview` — dùng với dữ liệu file từ hệ thống upload (`UploadFile | ClientUploadResponse`), tự xử lý lấy URL public từ API.
- `BaseFilePreview` — dùng với dữ liệu file đã chuẩn hóa (`FilePreviewItem[]`), hỗ trợ `getPublicUrl` tùy chỉnh và cache URL.

### Props chung

| Prop | Kiểu | Mặc định | Mô tả |
|------|------|----------|-------|
| `files` | `FilePreviewItem[]` hoặc `(UploadFile \| ClientUploadResponse)[]` | bắt buộc | Danh sách file cần xem trước |
| `activeIndex` | `number` | `0` (nội bộ) | Chỉ số file đang hiển thị (controlled) |
| `onChangeActiveIndex` | `(index: number) => void` | - | Callback khi chuyển file |
| `scaleStep` | `number` | `0.5` | Bước tăng/giảm mỗi lần zoom |
| `minScale` | `number` | `1` | Mức zoom nhỏ nhất |
| `maxScale` | `number` | `10` | Mức zoom lớn nhất |
| `onClose` | `() => void` | - | Callback khi đóng preview |

### Props riêng của `BaseFilePreview` (`FilePreviewItem`)

| Prop | Kiểu | Mô tả |
|------|------|-------|
| `id` | `string` | ID định danh file |
| `url` | `string` | URL trực tiếp của file |
| `ext` | `string` | Phần mở rộng file (ví dụ: `"pdf"`, `"png"`) |
| `fileName` | `string` | Tên hiển thị của file |
| `getPublicUrl` | `() => Promise<string>` | Hàm lấy URL public (tùy chọn, kết quả được cache) |

### Các loại file được hỗ trợ

| Loại | Định dạng |
|------|-----------|
| Hình ảnh | jpg, jpeg, png, svg, gif, bmp, tiff, tif, webp |
| Video | mp4, avi, mov, wmv, mkv, qt, webm |
| Âm thanh | mp3, aac, m4a |
| Tài liệu | pdf, doc, docx, xlsx, xls, csv, txt |

---

### RULE-FP-01: Chọn đúng variant `FilePreview` hoặc `BaseFilePreview`

Sử dụng `FilePreview` khi làm việc với dữ liệu file từ hệ thống upload nội bộ (UploadFile). Sử dụng `BaseFilePreview` khi bạn đã có dữ liệu file chuẩn hóa hoặc cần kiểm soát cách lấy URL public.

**Sai:**

```tsx
// ❌ Tự tạo object không đúng kiểu rồi truyền vào FilePreview
<FilePreview
  files={[{ id: "1", url: "https://example.com/file.png", ext: "png", fileName: "file.png" }]}
  onClose={onClose}
/>
```

**Đúng:**

```tsx
// ✅ Dữ liệu từ hệ thống upload — dùng FilePreview
<FilePreview files={uploadedFiles} onClose={onClose} />

// ✅ Dữ liệu đã chuẩn hóa — dùng BaseFilePreview
<BaseFilePreview
  files={[{ id: "1", url: "https://example.com/file.png", ext: "png", fileName: "file.png" }]}
  onClose={onClose}
/>
```

---

### RULE-FP-02: Render có điều kiện để mở/đóng FilePreview

`FilePreview` và `BaseFilePreview` không có prop `open`/`visible`. Component sử dụng `createPortal` để render vào `document.body`. Mount để hiển thị, unmount để ẩn.

**Sai:**

```tsx
// ❌ Không có prop open
<FilePreview open={isOpen} files={files} onClose={onClose} />
```

**Đúng:**

```tsx
// ✅ Dùng conditional rendering
{isOpen && <FilePreview files={files} onClose={onClose} />}
```

---

### RULE-FP-03: Sử dụng controlled `activeIndex` khi cần đồng bộ trạng thái bên ngoài

Nếu không truyền `activeIndex`, component tự quản lý index nội bộ (bắt đầu từ 0). Chỉ truyền `activeIndex` và `onChangeActiveIndex` khi cần kiểm soát hoặc đồng bộ trạng thái file đang xem từ bên ngoài.

**Sai:**

```tsx
// ❌ Truyền activeIndex mà không có onChangeActiveIndex — không thể chuyển file
<FilePreview files={files} activeIndex={0} onClose={onClose} />
```

**Đúng:**

```tsx
// ✅ Uncontrolled — component tự quản lý
<FilePreview files={files} onClose={onClose} />

// ✅ Controlled — đồng bộ với state bên ngoài
const [activeIndex, setActiveIndex] = useState(0);

<FilePreview
  files={files}
  activeIndex={activeIndex}
  onChangeActiveIndex={setActiveIndex}
  onClose={onClose}
/>
```

---

### RULE-FP-04: Không truyền rõ ràng các giá trị mặc định cho `scaleStep`, `minScale`, `maxScale`

Các prop zoom đã có giá trị mặc định hợp lý: `scaleStep = 0.5`, `minScale = 1`, `maxScale = 10`. Chỉ truyền khi cần tùy chỉnh khác giá trị mặc định.

**Sai:**

```tsx
// ❌ Thừa — các giá trị này đã là mặc định
<FilePreview
  files={files}
  scaleStep={0.5}
  minScale={1}
  maxScale={10}
  onClose={onClose}
/>
```

**Đúng:**

```tsx
// ✅ Bỏ qua khi dùng giá trị mặc định
<FilePreview files={files} onClose={onClose} />

// ✅ Chỉ truyền khi cần tùy chỉnh
<FilePreview files={files} scaleStep={0.25} maxScale={5} onClose={onClose} />
```

---

### RULE-FP-05: Sử dụng `getPublicUrl` trong `BaseFilePreview` cho file cần xác thực

Khi file yêu cầu URL có chữ ký (signed URL) hoặc cần gọi API để lấy URL public, truyền hàm `getPublicUrl` trong `FilePreviewItem`. Hook `useFilePublicUrl` sẽ tự động cache kết quả để tránh gọi API trùng lặp.

**Sai:**

```tsx
// ❌ Tự gọi API lấy URL trước khi truyền vào — không được cache
const publicUrl = await getSignedUrl(file.id);

<BaseFilePreview
  files={[{ id: file.id, url: publicUrl, ext: "pdf", fileName: "doc.pdf" }]}
  onClose={onClose}
/>
```

**Đúng:**

```tsx
// ✅ Truyền getPublicUrl để component tự xử lý và cache
<BaseFilePreview
  files={[{
    id: file.id,
    url: file.url,
    ext: "pdf",
    fileName: "doc.pdf",
    getPublicUrl: () => getSignedUrl(file.id),
  }]}
  onClose={onClose}
/>
```

---

### RULE-FP-06: Không tự xây dựng nút điều hướng hoặc zoom cho FilePreview

Component đã tích hợp sẵn nút Previous/Next, zoom in/out, reset zoom và hỗ trợ phím mũi tên trái/phải để điều hướng. Không cần tạo thêm UI điều khiển bên ngoài.

**Sai:**

```tsx
// ❌ Tự tạo nút điều hướng bên ngoài — trùng lặp với UI có sẵn
<div>
  <button onClick={() => setIndex(index - 1)}>Prev</button>
  <button onClick={() => setIndex(index + 1)}>Next</button>
  <FilePreview files={files} activeIndex={index} onChangeActiveIndex={setIndex} onClose={onClose} />
</div>
```

**Đúng:**

```tsx
// ✅ Component đã có sẵn nút điều hướng và zoom — chỉ cần render
<FilePreview files={files} onClose={onClose} />
```

---

### RULE-FP-07: Truyền mảng `files` chứa ít nhất một phần tử

Component truy cập `files[activeIndex]` trực tiếp. Truyền mảng rỗng sẽ gây lỗi runtime do `currentFile` là `undefined`.

**Sai:**

```tsx
// ❌ Mảng rỗng gây lỗi runtime
<FilePreview files={[]} onClose={onClose} />
```

**Đúng:**

```tsx
// ✅ Kiểm tra trước khi render
{files.length > 0 && <FilePreview files={files} onClose={onClose} />}
```
