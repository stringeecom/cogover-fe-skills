---
title: Cách sử dụng component MarkdownEditor trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến mất nội dung, upload ảnh thất bại, chế độ hiển thị không đúng, và xung đột chiều cao editor
tags: markdown-editor, markdown, editor, rich-text, upload, ui-kit
---

## Cách sử dụng component MarkdownEditor trong ui-kit

Đường dẫn component: `ui-kit/src/components/MarkdownEditor`

Props bắt buộc: `value`, `onChange`.
Props tùy chọn: `defaultValue`, `onBlur`, `onFocus`, `error`, `helperText`, `height`, `minHeight`, `maxHeight`, `maxHeightLongText`, `placeholder`, `color`, `readOnly`, `disabled`, `data-testid`, `defaultMode`, `showMenu`, `showMarkdown`, `showPreview`, `showFullScreen`, `plugins`, `handleImageUpload`, `classNameTextArea`, `tabIndex`.

Ref: forward ref tới `MdEditor` (từ `react-markdown-editor-lite`).

Giá trị mặc định: `defaultMode="both"`, `showMenu=true`, `showMarkdown=true`, `showPreview=true`, `showFullScreen=true`, `plugins=[]`.

---

### RULE-MKEDITOR-01: Luôn truyền `value` và `onChange` — MarkdownEditor là controlled component

`value` là markdown string, `onChange` nhận 3 tham số: `text` (markdown string), `html` (HTML string đã render), và `event` (optional). Không dùng uncontrolled mode.

**Sai:**

```tsx
// ❌ Thiếu onChange — editor không đồng bộ state
<MarkdownEditor value={markdown} />

// ❌ Thiếu value — editor không nhận nội dung ban đầu
<MarkdownEditor onChange={(text, html) => setMarkdown(text)} />
```

**Đúng:**

```tsx
// ✅
<MarkdownEditor value={markdown} onChange={(text, html) => setMarkdown(text)} />
```

---

### RULE-MKEDITOR-02: Sử dụng `minHeight`/`maxHeight` hoặc `height` cho chiều cao editor — không override CSS trực tiếp

Component hỗ trợ `height` (chiều cao cố định), `minHeight`/`maxHeight` (co giãn trong khoảng), và `maxHeightLongText` (giới hạn chiều cao tổng thể của editor). Không tự override CSS chiều cao.

**Sai:**

```tsx
// ❌ Override CSS trực tiếp — không tương thích với internal layout
<MarkdownEditor
  value={markdown}
  onChange={(text, html) => setMarkdown(text)}
  className={cx("[&_.rc-md-editor]:h-[300px]")}
/>
```

**Đúng:**

```tsx
// ✅ Chiều cao cố định
<MarkdownEditor value={markdown} onChange={(text, html) => setMarkdown(text)} height={300} />

// ✅ Co giãn với giới hạn
<MarkdownEditor
  value={markdown}
  onChange={(text, html) => setMarkdown(text)}
  minHeight={200}
  maxHeight={500}
/>

// ✅ Giới hạn chiều cao tổng thể khi nội dung dài
<MarkdownEditor
  value={markdown}
  onChange={(text, html) => setMarkdown(text)}
  maxHeightLongText={600}
/>
```

---

### RULE-MKEDITOR-03: Sử dụng `defaultMode` để chọn chế độ hiển thị — không ẩn panel bằng CSS

Component hỗ trợ 3 chế độ: `"both"` (markdown + preview), `"markdown"` (chỉ markdown), `"preview"` (chỉ preview). Không tự ẩn panel bằng CSS.

**Sai:**

```tsx
// ❌ Ẩn preview bằng CSS — panel vẫn tồn tại trong DOM
<MarkdownEditor
  value={markdown}
  onChange={(text, html) => setMarkdown(text)}
  className={cx("[&_.sec-html]:hidden")}
/>
```

**Đúng:**

```tsx
// ✅ Chỉ hiển thị markdown editor
<MarkdownEditor
  value={markdown}
  onChange={(text, html) => setMarkdown(text)}
  defaultMode="markdown"
/>

// ✅ Chỉ hiển thị preview
<MarkdownEditor
  value={markdown}
  onChange={(text, html) => setMarkdown(text)}
  defaultMode="preview"
/>

// ✅ Hiển thị cả hai (mặc định)
<MarkdownEditor
  value={markdown}
  onChange={(text, html) => setMarkdown(text)}
  defaultMode="both"
/>
```

---

### RULE-MKEDITOR-04: Sử dụng `showMenu`/`showMarkdown`/`showPreview`/`showFullScreen` để kiểm soát toolbar và panel — không ẩn bằng CSS

Các prop `showMenu`, `showMarkdown`, `showPreview`, `showFullScreen` kiểm soát việc hiển thị toolbar, nút chuyển chế độ markdown, nút chuyển chế độ preview, và nút full screen. Không tự ẩn các phần tử này bằng CSS.

**Sai:**

```tsx
// ❌ Ẩn toolbar bằng CSS — toolbar vẫn chiếm không gian trong DOM
<MarkdownEditor
  value={markdown}
  onChange={(text, html) => setMarkdown(text)}
  className={cx("[&_.rc-md-navigation]:hidden")}
/>
```

**Đúng:**

```tsx
// ✅ Ẩn toolbar
<MarkdownEditor
  value={markdown}
  onChange={(text, html) => setMarkdown(text)}
  showMenu={false}
/>

// ✅ Ẩn nút chuyển chế độ và full screen
<MarkdownEditor
  value={markdown}
  onChange={(text, html) => setMarkdown(text)}
  showMarkdown={false}
  showPreview={false}
  showFullScreen={false}
/>
```

---

### RULE-MKEDITOR-05: Sử dụng `handleImageUpload` để upload ảnh — không tự xử lý upload bên ngoài

Khi cần upload ảnh, truyền `handleImageUpload` — hàm nhận `File` và trả về `Promise<string>` (URL ảnh). Nếu không truyền, ảnh sẽ sử dụng `URL.createObjectURL` (blob URL tạm thời).

**Sai:**

```tsx
// ❌ Tự xử lý upload bên ngoài — không tích hợp với editor
const handlePaste = async (e) => {
  const file = e.clipboardData.files[0];
  const url = await uploadFile(file);
  setMarkdown((prev) => prev + `![image](${url})`);
};

<div onPaste={handlePaste}>
  <MarkdownEditor value={markdown} onChange={(text, html) => setMarkdown(text)} />
</div>
```

**Đúng:**

```tsx
// ✅ Upload ảnh qua prop handleImageUpload
<MarkdownEditor
  value={markdown}
  onChange={(text, html) => setMarkdown(text)}
  handleImageUpload={async (file) => {
    const formData = new FormData();
    formData.append("image", file);
    const response = await uploadApi(formData);
    return response.data.url;
  }}
/>
```

---

### RULE-MKEDITOR-06: Sử dụng `error` và `helperText` cho validation — không tự thêm HelperText bên ngoài

Component đã tích hợp `HelperText`. Khi `error=true`, border chuyển đỏ và `helperText` hiển thị với color error.

**Sai:**

```tsx
// ❌ Tự thêm HelperText bên ngoài — bị duplicate
<div>
  <MarkdownEditor value={markdown} onChange={(text, html) => setMarkdown(text)} />
  <HelperText color="error">Nội dung không được để trống</HelperText>
</div>
```

**Đúng:**

```tsx
// ✅
<MarkdownEditor
  value={markdown}
  onChange={(text, html) => setMarkdown(text)}
  error={hasError}
  helperText="Nội dung không được để trống"
/>
```

---

### RULE-MKEDITOR-07: Sử dụng `plugins` để thêm toolbar items — không tự build toolbar bên ngoài

Prop `plugins` nhận mảng tên plugin bổ sung. Danh sách plugin mặc định đã bao gồm: `header`, `font-bold`, `font-italic`, `font-underline`, `font-strikethrough`, `list-unordered`, `list-ordered`, `block-quote`, `block-wrap`, `block-code-inline`, `block-code-block`, `table`, `image`, `link`, `logger`, `mode-toggle`, `full-screen`, `tab-insert`. Plugins truyền vào sẽ được thêm vào sau danh sách mặc định.

**Sai:**

```tsx
// ❌ Tự build toolbar bên ngoài — không đồng bộ với editor
<div>
  <div className={cx("flex gap-[0.25rem]")}>
    <Button onClick={() => insertMarkdown("**bold**")}>Bold</Button>
    <Button onClick={() => insertMarkdown("_italic_")}>Italic</Button>
  </div>
  <MarkdownEditor value={markdown} onChange={(text, html) => setMarkdown(text)} />
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng plugins mặc định (đã bao gồm đầy đủ toolbar)
<MarkdownEditor value={markdown} onChange={(text, html) => setMarkdown(text)} />

// ✅ Thêm plugin bổ sung
<MarkdownEditor
  value={markdown}
  onChange={(text, html) => setMarkdown(text)}
  plugins={["custom-plugin"]}
/>
```

---

### RULE-MKEDITOR-08: Sử dụng `readOnly` để chặn chỉnh sửa — không tự disable bằng CSS hoặc overlay

Khi `readOnly=true`, toolbar bị ẩn, textarea không cho phép nhập, và background chuyển sang màu nhạt. Không tự tạo overlay hoặc dùng CSS `pointer-events: none`.

**Sai:**

```tsx
// ❌ Tự tạo overlay để chặn chỉnh sửa — không tương thích với keyboard navigation
<div className={cx("relative")}>
  <MarkdownEditor value={markdown} onChange={(text, html) => setMarkdown(text)} />
  {isReadOnly && <div className={cx("absolute inset-0")} />}
</div>
```

**Đúng:**

```tsx
// ✅
<MarkdownEditor
  value={markdown}
  onChange={(text, html) => setMarkdown(text)}
  readOnly={isReadOnly}
/>
```
