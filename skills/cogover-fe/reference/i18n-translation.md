---
title: Hướng dẫn Dịch i18n cho UI
tags: i18n, translation, react-i18next
---

# Hướng dẫn Dịch i18n cho UI

## Bước 1: Hỏi lại thông tin cần thiết

Khi được yêu cầu "dịch các text tiếng Việt", **LUÔN hỏi lại** 2 thông tin sau trước khi làm:

1. **Namespace** là gì? (ví dụ: `CHAT`, `CONTACT`, `COMMON`, ...)
2. **Prefix** là gì? (ví dụ: `contactDetail`, `chatMessage`, ...)

---

## Bước 2: Xác định loại dự án để import đúng

### Dự án router
```tsx
import { useTranslation, Trans } from 'react-i18next';
```

### Dự án ui-kit
Đọc file `src/hooks/i18n.ts` để xem cách định nghĩa hook, sau đó tạo hook tương tự.

### Các dự án còn lại (mặc định)
```tsx
import { useTranslation } from 'src/languages/global';
import Trans from 'src/languages/global/Trans';
```

---

## Bước 3: Lấy I18nNS từ namespace

Vào file `src/languages/i18n.ts` để lấy enum `I18nNS` tương ứng.

```tsx
const { t: tChat } = useTranslation(I18nNS.CHAT);
```

> Tên biến `t` đặt theo namespace: `tChat`, `tContact`, `tCommon`, ...

---

## Bước 4: Quy tắc dịch text

### 4.1 Chuỗi thông thường
- `"Xác nhận"` → `"Confirm"`

### 4.2 Chuỗi có placeholder
- `"Xin chào {{name}}"` → `"Hello {{name}}"`
> Placeholder dùng `{{}}` kép.

### 4.3 Template HTML
- Tiếng Việt: `"Đây là chữ <strong>bôi đậm</strong>: <em>{{name}}</em>"`
- Tiếng Anh: `"This is <1>bold</1> text: <2>{{name}}</2>"`

---

## Bước 5: Tạo bảng so sánh và in ra terminal

| key | Tiếng Việt | Tiếng Anh |
|-----|-----------|-----------|
| `confirm` | Xác nhận | Confirm |
| `greeting` | Xin chào {{name}} | Hello {{name}} |

- Cột **key**: camelCase của text tiếng Anh, **không** bao gồm prefix

---

## Bước 6: Sử dụng trong code

### 6.1 Chuỗi thông thường
```tsx
{tChat("contactDetail.confirm")}
```

### 6.2 Chuỗi có placeholder
```tsx
{tChat("contactDetail.greeting", { name: userName })}
```

### 6.3 Template HTML
```tsx
<Trans
  ns={I18nNS.CHAT}
  i18nKey="contactDetail.boldText"
  components={{ 1: <strong />, 2: <em /> }}
  values={{ name: userName }}
/>
```

> Số trong `components` tương ứng với số thứ tự tag: `<1>`, `<2>`, ...

---

## Lưu ý quan trọng

- **Không tạo file ngôn ngữ** (file JSON/translations). Chỉ cung cấp bảng dịch.
- **Không** đưa prefix vào cột `key` của bảng.
- Placeholder dùng `{{name}}` (ngoặc kép), không phải `{name}`.
- Số thứ tự trong `components` của `<Trans>` bắt đầu từ `1`.
