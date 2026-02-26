---
title: Cách sử dụng component CopyButton trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến logic clipboard trùng lặp, trạng thái tooltip bị lỗi và UX sao chép không nhất quán
tags: copy, clipboard, button, tooltip, ui-kit
---

## Cách sử dụng component CopyButton trong ui-kit

Đường dẫn component: `ui-kit/src/components/CopyButton`

Props: `value`, `icon`, `successIcon`, `showTooltip`, `onClick`, `className`. Hỗ trợ tất cả các `React.HTMLAttributes<HTMLButtonElement>` native.

### RULE-COPY-01: Sử dụng `CopyButton` thay vì logic clipboard thủ công

KHÔNG tự implement sao chép clipboard bằng `navigator.clipboard.writeText`. Luôn sử dụng `CopyButton` vì nó đã xử lý trạng thái sao chép, hoán đổi icon và tooltip tự động.

**Sai:**

```tsx
// ❌ Logic clipboard thủ công trùng lặp với những gì CopyButton đã xử lý
<button onClick={() => navigator.clipboard.writeText(value)}>
  <CopyIcon />
</button>
```

**Đúng:**

```tsx
// ✅
<CopyButton value={value} />
```

### RULE-COPY-02: Chỉ sử dụng `onClick` cho side effects, không dùng cho logic clipboard

`onClick` là một callback bổ sung — nó KHÔNG thay thế hành vi sao chép có sẵn. Không bao giờ đưa logic clipboard vào `onClick`.

**Sai:**

```tsx
// ❌ Logic copy trong onClick là thừa — CopyButton đã xử lý nó rồi
<CopyButton
  value={apiKey}
  onClick={() => navigator.clipboard.writeText(apiKey)}
/>
```

**Đúng:**

```tsx
// ✅ onClick chỉ dùng cho side effects
<CopyButton value={apiKey} onClick={() => trackEvent("api_key_copied")} />
```

### RULE-COPY-03: Chỉ sử dụng `showTooltip={false}` khi ngữ cảnh đã truyền đạt rõ hành động

Tooltip ("Copy" / "Đã sao chép") là cơ chế phản hồi chính. Chỉ tắt nó khi UI xung quanh đã cung cấp rõ phản hồi sao chép.

```tsx
// ✅ Tooltip bị tắt — label input xung quanh đã nói "Click to copy"
<CopyButton value={phoneNumber} showTooltip={false} />

// ✅ Mặc định — tooltip cung cấp phản hồi độc lập
<CopyButton value={url} />
```

### RULE-COPY-04: Không tự quản lý trạng thái copied bên ngoài component

`CopyButton` quản lý nội bộ trạng thái copied 3 giây và hoán đổi icon. Không bao giờ thêm state bên ngoài để phản ánh hành vi này.

**Sai:**

```tsx
// ❌ State bên ngoài trùng lặp logic nội bộ
const [copied, setCopied] = useState(false);
<CopyButton
  value={text}
  onClick={() => setCopied(true)}
  icon={copied ? <CheckIcon /> : <CopyIcon />}
/>;
```

**Đúng:**

```tsx
// ✅ Để component tự quản lý trạng thái copied của nó
<CopyButton value={text} />
```

### RULE-COPY-05: Sử dụng `icon` và `successIcon` để tùy chỉnh icons, không dùng children

`CopyButton` không chấp nhận children. Truyền custom icons thông qua `icon` (trạng thái mặc định) và `successIcon` (trạng thái đã sao chép).

**Sai:**

```tsx
// ❌ CopyButton không hỗ trợ children cho icons
<CopyButton value={text}>
  <MyIcon />
</CopyButton>
```

**Đúng:**

```tsx
// ✅
<CopyButton
  value={text}
  icon={<MyCustomCopyIcon />}
  successIcon={<MyCheckIcon />}
/>
```
