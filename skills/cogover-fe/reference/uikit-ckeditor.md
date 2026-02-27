---
title: Cách sử dụng component CKEditor trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến mất nội dung, upload ảnh thất bại, autocomplete không hoạt động, và xung đột dark mode
tags: ckeditor, rich-text, editor, wysiwyg, upload, autocomplete, ui-kit
---

## Cách sử dụng component CKEditor trong ui-kit

Đường dẫn component: `ui-kit/src/components/CKEditor`

Props bắt buộc: `value`, `onChange`.
Props tùy chọn: `name`, `onBlur`, `onFocus`, `error`, `helperText`, `height`, `minHeight`, `maxHeight`, `placeholder`, `className`, `readOnly`, `uploadImageUrl`, `stickyTopToolbar`, `processFileUploadRequest`, `getUrlFromFileUploadResponse`, `autocompleteConfigs`, `disabledModules`, `tabIndex`, `autofocus`, `autoGrow`.

Ref methods: `setTextValue(value: string)`, `getEditor()`.

Giá trị mặc định: `enterMode=ENTER_DIV`, `shiftEnterMode=ENTER_BR`, font `Nunito 14px`.

---

### RULE-CKEDITOR-01: Luôn truyền `value` và `onChange` — CKEditor là controlled component

`value` là HTML string, `onChange` nhận HTML string đã được xử lý (hex→rgb, trim whitespace). Không dùng uncontrolled mode.

**Sai:**

```tsx
// ❌ Thiếu onChange — editor không đồng bộ state
<CKEditor value={html} />

// ❌ Thiếu value — editor không nhận nội dung ban đầu
<CKEditor onChange={setHtml} />
```

**Đúng:**

```tsx
// ✅
<CKEditor value={html} onChange={setHtml} />
```

---

### RULE-CKEDITOR-02: Sử dụng `minHeight`/`maxHeight` hoặc `autoGrow` cho editor co giãn — không override CSS height

Khi truyền `minHeight`, `maxHeight` hoặc `autoGrow`, plugin `autogrow` tự kích hoạt. Không tự override chiều cao bằng CSS.

**Sai:**

```tsx
// ❌ Override CSS — không kích hoạt plugin autogrow
<CKEditor
  value={html}
  onChange={setHtml}
  className={cx("[&_iframe]:min-h-[200px]")}
/>
```

**Đúng:**

```tsx
// ✅ Chiều cao cố định
<CKEditor value={html} onChange={setHtml} height={300} />

// ✅ Co giãn với giới hạn
<CKEditor value={html} onChange={setHtml} minHeight={200} maxHeight={500} />

// ✅ Co giãn không giới hạn
<CKEditor value={html} onChange={setHtml} autoGrow />
```

---

### RULE-CKEDITOR-03: Truyền đủ `uploadImageUrl` + `getUrlFromFileUploadResponse` để upload ảnh hoạt động

Upload ảnh yêu cầu cả URL endpoint và hàm trích xuất URL từ response. Thiếu một trong hai sẽ khiến upload không hoạt động hoặc ảnh không hiển thị.

**Sai:**

```tsx
// ❌ Thiếu getUrlFromFileUploadResponse — upload thành công nhưng không lấy được URL ảnh
<CKEditor
  value={html}
  onChange={setHtml}
  uploadImageUrl="/api/upload"
/>
```

**Đúng:**

```tsx
// ✅
<CKEditor
  value={html}
  onChange={setHtml}
  uploadImageUrl="/api/upload"
  getUrlFromFileUploadResponse={(response) => response.data.url}
/>

// ✅ Tuỳ chỉnh FormData khi upload
<CKEditor
  value={html}
  onChange={setHtml}
  uploadImageUrl="/api/upload"
  processFileUploadRequest={(file) => {
    const formData = new FormData();
    formData.append("image", file);
    return formData;
  }}
  getUrlFromFileUploadResponse={(response) => response.data.url}
/>
```

---

### RULE-CKEDITOR-04: Cấu hình `autocompleteConfigs` đầy đủ `pattern` và `dataCallback` — không tự build autocomplete

Mỗi config cần ít nhất `pattern` (RegExp) và `dataCallback` (trả về danh sách suggestions). Sử dụng `toolbarButton` nếu cần nút trên toolbar.

**Sai:**

```tsx
// ❌ Tự build autocomplete bên ngoài CKEditor
<div>
  <CKEditor value={html} onChange={setHtml} />
  {showSuggestions && <SuggestionList />}
</div>
```

**Đúng:**

```tsx
// ✅ @mention autocomplete
<CKEditor
  value={html}
  onChange={setHtml}
  autocompleteConfigs={[
    {
      pattern: /@(\w*)$/,
      dataCallback: async ({ query }) => {
        const users = await fetchUsers(query);
        return users.map((u) => ({ id: u.id, name: u.name }));
      },
      outputTemplate: '<span class="mention">@{name}</span>',
      toolbarButton: {
        buttonKey: "mention",
        buttonText: "@",
        insertedText: "@",
      },
    },
  ]}
/>
```

---

### RULE-CKEDITOR-05: Sử dụng ref `setTextValue` để chèn text — không truy cập DOM trực tiếp

`setTextValue` chèn text tại vị trí cursor (nếu editor đã từng focus) hoặc append vào cuối. Không tự manipulate DOM của iframe.

**Sai:**

```tsx
// ❌ Truy cập DOM trực tiếp — dễ vỡ, không đồng bộ với CKEditor state
const iframe = document.querySelector(".cke_wysiwyg_frame");
iframe.contentDocument.body.innerHTML += "<p>New text</p>";
```

**Đúng:**

```tsx
// ✅
const editorRef = useRef<CKEditorRef>(null);

<CKEditor ref={editorRef} value={html} onChange={setHtml} />

<Button onClick={() => editorRef.current?.setTextValue("Inserted text")}>
  Insert
</Button>
```

---

### RULE-CKEDITOR-06: Sử dụng `error` và `helperText` cho validation — không tự thêm HelperText bên ngoài

Component đã tích hợp `HelperText`. Khi `error=true`, border chuyển đỏ và `helperText` hiển thị với color error.

**Sai:**

```tsx
// ❌ Tự thêm HelperText bên ngoài — bị duplicate
<div>
  <CKEditor value={html} onChange={setHtml} />
  <HelperText color="error">Nội dung không được để trống</HelperText>
</div>
```

**Đúng:**

```tsx
// ✅
<CKEditor
  value={html}
  onChange={setHtml}
  error={hasError}
  helperText="Nội dung không được để trống"
/>
```

---

### RULE-CKEDITOR-07: Sử dụng `stickyTopToolbar` khi editor nằm trong scrollable container

Truyền offset (px) để toolbar sticky khi scroll. Không tự viết CSS sticky cho toolbar.

**Sai:**

```tsx
// ❌ Tự viết CSS sticky — không tương thích với CKEditor internal layout
<CKEditor
  value={html}
  onChange={setHtml}
  className={cx("[&_.cke_top]:sticky [&_.cke_top]:top-0")}
/>
```

**Đúng:**

```tsx
// ✅ Toolbar sticky cách top 64px (ví dụ: dưới header)
<CKEditor value={html} onChange={setHtml} stickyTopToolbar={64} />
```

---

### RULE-CKEDITOR-08: Sử dụng `disabledModules` để tắt toolbar items — không ẩn bằng CSS

Truyền mảng tên module cần tắt. Không tự ẩn toolbar buttons bằng CSS override.

**Sai:**

```tsx
// ❌ Ẩn bằng CSS — nút vẫn tồn tại, có thể access qua keyboard
<CKEditor
  value={html}
  onChange={setHtml}
  className={cx("[&_.cke_button__source]:hidden")}
/>
```

**Đúng:**

```tsx
// ✅
<CKEditor
  value={html}
  onChange={setHtml}
  disabledModules={["Source", "Maximize"]}
/>
```
