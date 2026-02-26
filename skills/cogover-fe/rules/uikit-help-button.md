---
title: Cách sử dụng component HelpButton trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến vấn đề render raw HTML, wrappers popper thừa và UX help không nhất quán
tags: help, tooltip, popper, rich-text, ui-kit
---

## Cách sử dụng component HelpButton trong ui-kit

Đường dẫn component: `ui-kit/src/components/HelpButton`

Props: `helpText`, `renderHTML`, `element`, `classNameContainer`, `popperProps`. Hỗ trợ tất cả `React.ButtonHTMLAttributes<HTMLButtonElement>` native.

Render một nút icon tròn `ⓘ` mở một popover nổi với nội dung help khi click.

### RULE-HELP-01: Sử dụng `HelpButton` cho tất cả nội dung help/hint inline, không dùng tooltips tùy chỉnh

Khi hiển thị help hoặc hint text ngữ cảnh bên cạnh một field hoặc label, luôn sử dụng `HelpButton`. KHÔNG tự xây dựng kết hợp info icon + tooltip tùy chỉnh.

**Sai:**
```tsx
// ❌ Info icon tùy chỉnh trùng lặp hành vi HelpButton
<Tooltip content="Nhập tên đầy đủ của bạn">
  <InfoIcon />
</Tooltip>
```

**Đúng:**
```tsx
// ✅
<HelpButton helpText="Nhập tên đầy đủ của bạn" />
```

### RULE-HELP-02: Sử dụng `renderHTML={true}` khi `helpText` chứa rich text từ Quill editor

Khi `helpText` là một chuỗi HTML rich-text từ Quill (từ một rich editor), set `renderHTML={true}`. Nó sẽ được render qua `RenderRichEditorValue`. KHÔNG sử dụng `dangerouslySetInnerHTML` trực tiếp.

**Sai:**
```tsx
// ❌ Không an toàn và bỏ qua việc render rich editor
<HelpButton helpText={<span dangerouslySetInnerHTML={{ __html: richText }} />} />
```

**Đúng:**
```tsx
// ✅ renderHTML truyền helpText qua RenderRichEditorValue
<HelpButton helpText={field.helpText} renderHTML />
```

### RULE-HELP-03: Truyền plain `ReactNode` vào `helpText` cho nội dung text đơn giản

Khi `helpText` là plain text hoặc JSX đơn giản, KHÔNG set `renderHTML`. Mặc định sẽ render `helpText` trực tiếp như ReactNode.

```tsx
// ✅ Plain text
<HelpButton helpText="Mã này được dùng để xác định ticket trong hệ thống." />

// ✅ JSX với định dạng
<HelpButton
  helpText={
    <span>
      Tối đa <strong>500</strong> ký tự.
    </span>
  }
/>
```

### RULE-HELP-04: Sử dụng `popperProps` để override placement hoặc offset, không re-wrap với Popper

`HelpButton` được xây dựng trên `Popper` nội bộ. Để thay đổi placement hoặc offset, sử dụng `popperProps`. KHÔNG wrap `HelpButton` bên trong một `Popper` khác.

**Sai:**
```tsx
// ❌ Wrapping thêm một lớp Popper thứ hai
<Popper placement="top" render={() => <HelpButton helpText="..." />}>
  {(params) => <div {...params} />}
</Popper>
```

**Đúng:**
```tsx
// ✅ Override qua popperProps
<HelpButton
  helpText="Thông tin bổ sung"
  popperProps={{ placement: "top-start", offset: [0, 8] }}
/>
```

### RULE-HELP-05: Sử dụng `classNameContainer` để style hộp popover, không dùng `className`

`className` áp dụng cho nút trigger. Để style container nội dung popover (background, padding, border, v.v.), sử dụng `classNameContainer`.

**Sai:**
```tsx
// ❌ className target nút, không phải popover
<HelpButton helpText="..." className="max-w-[400px] bg-gray-1" />
```

**Đúng:**
```tsx
// ✅ classNameContainer target hộp nội dung popover
<HelpButton helpText="..." classNameContainer="max-w-[25rem]" />
```
