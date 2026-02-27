## Cách sử dụng component CKEditor trong ui-kit

Đường dẫn: `ui-kit/src/components/CKEditor`

Props bắt buộc: `value` (HTML string), `onChange` (nhận HTML string đã xử lý).
Props tùy chọn: `name`, `onBlur`, `onFocus`, `error`, `helperText`, `height`, `minHeight`, `maxHeight`, `placeholder`, `className`, `readOnly`, `uploadImageUrl`, `stickyTopToolbar`, `processFileUploadRequest`, `getUrlFromFileUploadResponse`, `autocompleteConfigs`, `disabledModules`, `tabIndex`, `autofocus`, `autoGrow`.

Ref methods: `setTextValue(value: string)`, `getEditor()`.
Defaults: `enterMode=ENTER_DIV`, `shiftEnterMode=ENTER_BR`, font Nunito 14px.

```tsx
// Cơ bản (controlled)
<CKEditor value={html} onChange={setHtml} />

// Chiều cao: cố định, co giãn có giới hạn, hoặc không giới hạn
<CKEditor value={html} onChange={setHtml} height={300} />
<CKEditor value={html} onChange={setHtml} minHeight={200} maxHeight={500} />
<CKEditor value={html} onChange={setHtml} autoGrow />

// Upload ảnh (cần cả hai props)
<CKEditor
  value={html} onChange={setHtml}
  uploadImageUrl="/api/upload"
  getUrlFromFileUploadResponse={(response) => response.data.url}
/>

// Validation
<CKEditor value={html} onChange={setHtml} error={hasError} helperText="Nội dung không được để trống" />

// Autocomplete (@mention)
<CKEditor
  value={html} onChange={setHtml}
  autocompleteConfigs={[{
    pattern: /@(\w*)$/,
    dataCallback: async ({ query }) => fetchUsers(query).then(u => u.map(x => ({ id: x.id, name: x.name }))),
    outputTemplate: '<span class="mention">@{name}</span>',
    toolbarButton: { buttonKey: "mention", buttonText: "@", insertedText: "@" },
  }]}
/>

// Chèn text qua ref
const editorRef = useRef<CKEditorRef>(null);
<CKEditor ref={editorRef} value={html} onChange={setHtml} />
<Button onClick={() => editorRef.current?.setTextValue("text")}>Insert</Button>

// Toolbar sticky (offset px từ top)
<CKEditor value={html} onChange={setHtml} stickyTopToolbar={64} />

// Tắt modules
<CKEditor value={html} onChange={setHtml} disabledModules={["Source", "Maximize"]} />
```

**DO:** Luôn truyền cả `value` và `onChange`. Dùng `minHeight`/`maxHeight`/`autoGrow` cho chiều cao. Truyền đủ `uploadImageUrl` + `getUrlFromFileUploadResponse` cho upload. Dùng `error`+`helperText` cho validation. Dùng ref `setTextValue` để chèn text. Dùng `stickyTopToolbar` cho container scrollable. Dùng `disabledModules` để tắt toolbar items.

**DON'T:** Bỏ `onChange` hoặc `value`. Override CSS height. Tự build autocomplete bên ngoài. Thêm HelperText bên ngoài. Truy cập DOM iframe trực tiếp. Ẩn toolbar bằng CSS.
