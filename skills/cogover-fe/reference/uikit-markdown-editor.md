## Cách sử dụng component MarkdownEditor trong ui-kit

Path: `ui-kit/src/components/MarkdownEditor`

Required props: `value`, `onChange`.

Optional props: `defaultValue`, `onBlur`, `onFocus`, `error`, `helperText`, `height`, `minHeight`, `maxHeight`, `maxHeightLongText`, `placeholder`, `color`, `readOnly`, `disabled`, `data-testid`, `defaultMode`, `showMenu`, `showMarkdown`, `showPreview`, `showFullScreen`, `plugins`, `handleImageUpload`, `classNameTextArea`, `tabIndex`.

Defaults: `defaultMode="both"`, `showMenu=true`, `showMarkdown=true`, `showPreview=true`, `showFullScreen=true`, `plugins=[]`.

`onChange` signature: `(text: string, html: string, event?: Event) => void`

---

### Usage

```tsx
// Basic — controlled component, always pass value and onChange
const [markdown, setMarkdown] = useState("");

<MarkdownEditor value={markdown} onChange={(text, html) => setMarkdown(text)} />

// Fixed height
<MarkdownEditor value={markdown} onChange={(text, html) => setMarkdown(text)} height={300} />

// Min/max height (auto-expand)
<MarkdownEditor value={markdown} onChange={(text, html) => setMarkdown(text)} minHeight={200} maxHeight={500} />

// Display modes
<MarkdownEditor value={markdown} onChange={(text, html) => setMarkdown(text)} defaultMode="markdown" />
<MarkdownEditor value={markdown} onChange={(text, html) => setMarkdown(text)} defaultMode="preview" />

// Hide toolbar elements
<MarkdownEditor value={markdown} onChange={(text, html) => setMarkdown(text)} showMenu={false} />
<MarkdownEditor value={markdown} onChange={(text, html) => setMarkdown(text)} showMarkdown={false} showPreview={false} showFullScreen={false} />

// Image upload — returns Promise<url>
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

// Error state
<MarkdownEditor
  value={markdown}
  onChange={(text, html) => setMarkdown(text)}
  error={hasError}
  helperText="Nội dung không được để trống"
/>

// Read-only — hides toolbar, disables textarea
<MarkdownEditor value={markdown} onChange={(text, html) => setMarkdown(text)} readOnly={isReadOnly} />
```

---

### DO / DON'T

```tsx
// ❌ Missing onChange — editor won't sync state
<MarkdownEditor value={markdown} />

// ❌ Override CSS directly for height
<MarkdownEditor value={markdown} onChange={...} className={cx("[&_.rc-md-editor]:h-[300px]")} />

// ❌ Hide panels via CSS — use defaultMode instead
<MarkdownEditor value={markdown} onChange={...} className={cx("[&_.sec-html]:hidden")} />

// ❌ Build toolbar outside — use plugins prop for extensions
<div>
  <Button onClick={() => insertMarkdown("**bold**")}>Bold</Button>
  <MarkdownEditor value={markdown} onChange={...} />
</div>

// ❌ Build overlay for readOnly — use readOnly prop
<div className="relative">
  <MarkdownEditor value={markdown} onChange={...} />
  {isReadOnly && <div className="absolute inset-0" />}
</div>

// ❌ Add HelperText externally — already built-in
<div>
  <MarkdownEditor value={markdown} onChange={...} />
  <HelperText color="error">Error message</HelperText>
</div>
```
