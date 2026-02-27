## Cách sử dụng FileInput Component của ui-kit

Đường dẫn: `ui-kit/src/components/FileInput`

| Prop | Kiểu | Bắt buộc | Mô tả |
|------|------|----------|-------|
| `isMaxFile` | `boolean` | Có | Khi `true`, component không render |
| `canEdit` | `boolean` | Có | Cho phép chỉnh sửa |
| `onChange` | `(e: ChangeEvent<HTMLInputElement>) => void` | Có | Callback khi chọn file |
| `tooltip` | `string` | Không | Danh sách định dạng, hiển thị trong Tooltip nội bộ |
| `maxSize` | `number` | Không | Dung lượng tối đa (bytes), hiển thị trong Tooltip |
| `acceptTypes` | `string[]` | Không | Phần mở rộng không có dấu chấm: `["pdf", "docx"]` |
| `multipleFile` | `boolean` | Không | Cho phép chọn nhiều file |
| `iconRight` | `ReactNode` | Không | Nội dung bên phải nút chọn file |
| `disabled` | `boolean` | Không | Vô hiệu hóa |
| `className` | `string` | Không | Class cho container |

```tsx
<FileInput
  isMaxFile={isMaxFile}        // component tự ẩn khi true, không cần conditional render ngoài
  canEdit={true}
  tooltip="PDF, DOCX, PNG"
  maxSize={5242880}            // bytes
  acceptTypes={["pdf", "docx", "png"]}  // không có dấu chấm
  multipleFile={true}
  iconRight={<SmartPasteButton onClick={handleSmartPaste} />}
  onChange={(e) => {
    const files = Array.from(e.target.files ?? []);
    processFiles(files);
    e.target.value = "";       // reset để có thể chọn lại cùng file
  }}
/>
```

**DO:** Truyền `isMaxFile` trực tiếp (component tự return null). Dùng `tooltip`+`maxSize` cho Tooltip nội bộ. Dùng `acceptTypes` không có dấu chấm. Dùng `iconRight` để thêm nội dung bên phải nút. Reset `e.target.value = ""` sau khi xử lý.

**DON'T:** Tự tạo `<input type="file">`. Conditional render bên ngoài dựa vào `isMaxFile`. Wrap Tooltip bên ngoài. Truyền MIME type hoặc extension có dấu chấm vào `acceptTypes`. Xử lý nhiều file mà không truyền `multipleFile`.
