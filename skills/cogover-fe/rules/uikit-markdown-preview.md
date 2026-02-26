---
title: Cách sử dụng MarkdownPreview Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng MarkdownPreview sai dẫn đến việc render markdown không đúng cách, truyền HTML thay vì markdown, và tuỳ chỉnh style không đúng chỗ
tags: markdown, preview, markdown-preview, react-markdown, ui-kit
---

## Cách sử dụng MarkdownPreview Component của ui-kit

Đường dẫn component: `ui-kit/src/components/MarkdownPreview`

Props: `content`, `className`, `style`, `onClick`.

Component sử dụng `react-markdown` với các plugin: `remarkGfm` (GitHub Flavored Markdown), `remarkToc` (Table of Contents), `rehypeRaw` (hỗ trợ HTML trong markdown), `rehypeSlug` (thêm id cho heading), `rehypeHighlight` (syntax highlighting cho code block).

---

### RULE-MDP-01: Truyền nội dung markdown qua prop `content`, không dùng `dangerouslySetInnerHTML`

`MarkdownPreview` đã tích hợp sẵn `react-markdown` với `rehypeRaw` để render cả markdown lẫn HTML nhúng trong markdown. Không cần tự xử lý HTML.

**Sai:**

```tsx
// ❌ Tự render HTML thay vì dùng MarkdownPreview
<div dangerouslySetInnerHTML={{ __html: markdownContent }} />
```

**Đúng:**

```tsx
// ✅ Truyền nội dung markdown qua prop content
<MarkdownPreview content={markdownContent} />
```

---

### RULE-MDP-02: Luôn đảm bảo `content` không phải `null` hoặc `undefined`

Prop `content` có kiểu `string`. Component gọi `content.trim()` nội bộ, nên nếu truyền `null` hoặc `undefined` sẽ gây lỗi runtime.

**Sai:**

```tsx
// ❌ content có thể là undefined — gây lỗi runtime khi gọi .trim()
<MarkdownPreview content={item.defaultValue} />
```

**Đúng:**

```tsx
// ✅ Đảm bảo content luôn là string bằng fallback
<MarkdownPreview content={item.defaultValue ?? ""} />

// ✅ Hoặc dùng giá trị mặc định
<MarkdownPreview content={value ?? ""} />
```

---

### RULE-MDP-03: Sử dụng `className` để tuỳ chỉnh style container, không override class nội bộ

`className` được áp dụng cho `div` container bên ngoài (đã có sẵn class `markdown-body`). Không dùng selector phức tạp để override style nội bộ của `react-markdown`.

**Sai:**

```tsx
// ❌ Override style nội bộ bằng selector phức tạp
<MarkdownPreview
  content={value}
  className="[&_.markdown-body]:bg-white [&_div_.text-typo-primary]:text-red-500"
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng className cho container (ví dụ: thêm padding, overflow, background)
<MarkdownPreview
  content={value}
  className={cx(
    "overflow-auto",
    "p-[0.75rem] rounded",
    "!bg-primary-light-94",
  )}
/>
```

---

### RULE-MDP-04: Sử dụng prop `style` cho các giá trị động, không inline trong `className`

Khi cần thiết lập kích thước động (minHeight, maxHeight) dựa trên biến, sử dụng prop `style` thay vì tạo class Tailwind động.

**Sai:**

```tsx
// ❌ Tailwind không hỗ trợ giá trị động từ biến JS
<MarkdownPreview
  content={value}
  className={`min-h-[${minHeight}px] max-h-[${maxHeight}px]`}
/>
```

**Đúng:**

```tsx
// ✅ Dùng prop style cho giá trị động
<MarkdownPreview
  content={value}
  style={{
    minHeight: `${minHeight}px`,
    maxHeight: `${maxHeight}px`,
  }}
/>
```

---

### RULE-MDP-05: Không tự cài đặt và cấu hình `react-markdown` khi đã có `MarkdownPreview`

`MarkdownPreview` đã tích hợp đầy đủ các plugin cần thiết: GFM (bảng, checkbox, strikethrough), Table of Contents, syntax highlighting, và hỗ trợ HTML nhúng. Không cần tự import `react-markdown` và cấu hình lại.

**Sai:**

```tsx
// ❌ Tự cấu hình react-markdown — duplicate logic đã có trong MarkdownPreview
import Markdown from "react-markdown";
import remarkGfm from "remark-gfm";
import rehypeHighlight from "rehype-highlight";

<Markdown remarkPlugins={[remarkGfm]} rehypePlugins={[rehypeHighlight]}>
  {content}
</Markdown>
```

**Đúng:**

```tsx
// ✅ Sử dụng MarkdownPreview đã có sẵn đầy đủ plugin
import { MarkdownPreview } from "ui-kit";

<MarkdownPreview content={content} />
```

---

### RULE-MDP-06: Phân biệt khi nào dùng `MarkdownPreview` và khi nào dùng `RenderRichEditorValue`

`MarkdownPreview` dùng để render nội dung **markdown**. `RenderRichEditorValue` dùng để render nội dung **HTML** từ rich text editor (CKEditor). Không dùng lẫn lộn.

**Sai:**

```tsx
// ❌ Dùng MarkdownPreview để render HTML từ CKEditor
<MarkdownPreview content={htmlFromEditor} />

// ❌ Dùng RenderRichEditorValue để render markdown
<RenderRichEditorValue value={markdownContent} />
```

**Đúng:**

```tsx
// ✅ Dùng MarkdownPreview cho nội dung markdown
if (displayType === "markdown") {
  return <MarkdownPreview content={item.content ?? ""} />;
}

// ✅ Dùng RenderRichEditorValue cho nội dung HTML
if (displayType === "html") {
  return <RenderRichEditorValue value={item.content ?? ""} />;
}
```

---

## Tham chiếu style

| Phần tử | Class mặc định |
|---------|----------------|
| Container ngoài | `markdown-body !bg-background-default !font-[inherit]` |
| Nội dung Markdown | `text-typo-primary` |
| Danh sách `<ul>` | `list-disc` |
| Code block `<pre>` | `!bg-primary-light-80` |
| Hàng bảng `<tr>` | `!bg-primary-light-80` |

## Các plugin đã tích hợp

| Plugin | Chức năng |
|--------|-----------|
| `remarkGfm` | Hỗ trợ GitHub Flavored Markdown (bảng, checkbox, strikethrough, autolink) |
| `remarkToc` | Tự động tạo Table of Contents (maxDepth: 2) |
| `rehypeRaw` | Cho phép render HTML nhúng trong markdown |
| `rehypeSlug` | Thêm id tự động cho các heading |
| `rehypeHighlight` | Syntax highlighting cho code block |
