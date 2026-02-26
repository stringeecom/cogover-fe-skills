---
title: Cách sử dụng component CopyLinkButton trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến URL bị sai, trạng thái copied không nhất quán và UX sao chép link kém
tags: copy, clipboard, link, url, button, ui-kit
---

## Cách sử dụng component CopyLinkButton trong ui-kit

Đường dẫn component: `ui-kit/src/components/CopyLinkButton`

Props: `link` (bắt buộc). Component render một `Button` variant `text` size `small` với icon copy/check.

### RULE-COPYLINK-01: Sử dụng `CopyLinkButton` thay vì tự xây dựng logic sao chép URL

KHÔNG tự implement logic sao chép URL bằng `navigator.clipboard.writeText(window.location.origin + path)`. Luôn sử dụng `CopyLinkButton` vì nó đã xử lý ghép URL, trạng thái copied, hoán đổi icon và auto-reset sau 5 giây.

**Sai:**

```tsx
// ❌ Logic sao chép URL thủ công trùng lặp với CopyLinkButton
const [copied, setCopied] = useState(false);
<button
  onClick={() => {
    navigator.clipboard.writeText(window.location.origin + "/records/123");
    setCopied(true);
    setTimeout(() => setCopied(false), 5000);
  }}
>
  {copied ? <CheckIcon /> : <CopyIcon />}
</button>;
```

**Đúng:**

```tsx
// ✅
<CopyLinkButton link="/records/123" />
```

### RULE-COPYLINK-02: Truyền đường dẫn tương đối cho prop `link`, không truyền full URL

`CopyLinkButton` tự động ghép `window.location.origin` với giá trị `link`. Nếu truyền full URL sẽ bị ghép sai.

**Sai:**

```tsx
// ❌ Full URL sẽ bị ghép thành "https://app.example.com/https://app.example.com/records/123"
<CopyLinkButton link="https://app.example.com/records/123" />
```

**Đúng:**

```tsx
// ✅ Đường dẫn tương đối — kết quả: "https://app.example.com/records/123"
<CopyLinkButton link="/records/123" />
```

### RULE-COPYLINK-03: Không tự quản lý trạng thái copied bên ngoài component

`CopyLinkButton` quản lý nội bộ trạng thái copied với auto-reset 5 giây và hoán đổi icon (copy → check circle). Không thêm state bên ngoài để phản ánh hành vi này.

**Sai:**

```tsx
// ❌ State bên ngoài trùng lặp logic nội bộ
const [copied, setCopied] = useState(false);
<CopyLinkButton link="/records/123" />;
{
  copied && <span>Đã sao chép!</span>;
}
```

**Đúng:**

```tsx
// ✅ Để component tự quản lý trạng thái — icon tự đổi sang check circle khi copied
<CopyLinkButton link="/records/123" />
```

### RULE-COPYLINK-04: Phân biệt `CopyButton` và `CopyLinkButton`

`CopyButton` dùng để sao chép một giá trị bất kỳ (text, code, API key). `CopyLinkButton` chuyên dùng để sao chép URL đầy đủ từ đường dẫn tương đối. Không dùng lẫn lộn.

**Sai:**

```tsx
// ❌ Dùng CopyButton rồi tự ghép URL
<CopyButton value={window.location.origin + "/records/123"} />
```

**Đúng:**

```tsx
// ✅ Dùng CopyLinkButton cho link — tự ghép origin
<CopyLinkButton link="/records/123" />

// ✅ Dùng CopyButton cho giá trị không phải URL
<CopyButton value={apiKey} />
```
