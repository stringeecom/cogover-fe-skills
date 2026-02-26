---
title: Cách sử dụng component HelperText trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến màu phản hồi form không nhất quán, typography sai và manual color overrides phá vỡ design system
tags: helper-text, form, error, warning, success, validation, ui-kit
---

## Cách sử dụng component HelperText trong ui-kit

Đường dẫn component: `ui-kit/src/components/HelperText`

Props: `color`, `className`. Hỗ trợ tất cả `React.HTMLAttributes<HTMLElement>` native. Hỗ trợ forwarding `ref` đến phần tử `<p>`.

Render một tag `<p>` với typography `prose-caption2`, màu `text-typo-secondary` và spacing `mt-[4px]`. Sử dụng `color` để chuyển sang các màu phản hồi ngữ nghĩa.

### RULE-HELPER-01: Sử dụng `HelperText` cho tất cả các tin nhắn helper/error/warning của form field

KHÔNG sử dụng tag `<p>`, `<span>` hoặc `<small>` thô cho các tin nhắn phản hồi form. Luôn sử dụng `HelperText` để đảm bảo typography, spacing và màu nhất quán.

**Sai:**

```tsx
// ❌ Tag thô mất typography và spacing rules
<p className={cx("text-error text-xs mt-1")}>Email không hợp lệ</p>
<span className={cx("text-warning")}>Trường này sắp hết hạn</span>
```

**Đúng:**

```tsx
// ✅
<HelperText color="error">Email không hợp lệ</HelperText>
<HelperText color="warning">Trường này sắp hết hạn</HelperText>
```

### RULE-HELPER-02: Sử dụng prop `color` cho phản hồi ngữ nghĩa, không bao giờ override màu qua `className`

Prop `color` map đến design-system tokens (`text-error`, `text-warning`, `text-success`). KHÔNG override màu text thủ công qua `className`.

**Sai:**

```tsx
// ❌ Màu hardcode bỏ qua design system tokens
<HelperText className={cx("text-red-500")}>Lỗi nhập liệu</HelperText>
```

**Đúng:**

```tsx
// ✅
<HelperText color="error">Lỗi nhập liệu</HelperText>
<HelperText color="success">Đã lưu thành công</HelperText>
<HelperText color="warning">Cảnh báo: dữ liệu chưa được xác nhận</HelperText>
```

### RULE-HELPER-03: Bỏ qua `color` cho helper text trung lập/secondary

Khi tin nhắn là thông tin (không có trạng thái ngữ nghĩa), bỏ qua `color`. Style mặc định (`text-typo-secondary`) là đúng cho các gợi ý trung lập.

```tsx
// ✅ Gợi ý trung lập — không cần color
<HelperText>Tối đa 255 ký tự.</HelperText>
<HelperText>Định dạng: DD/MM/YYYY</HelperText>
```

### RULE-HELPER-04: Không thêm `mt-*` margin thủ công — `HelperText` đã có `mt-[4px]`

Component có margin top `mt-[4px]` sẵn. Thêm margin bổ sung qua `className` gây khoảng cách kép.

**Sai:**

```tsx
// ❌ Margin top kép
<HelperText className={cx("mt-[0.25rem]")} color="error">
  Trường bắt buộc
</HelperText>
```

**Đúng:**

```tsx
// ✅ Không cần margin bổ sung
<HelperText color="error">Trường bắt buộc</HelperText>
```

### RULE-HELPER-05: Render có điều kiện `HelperText`, không truyền chuỗi rỗng

KHÔNG render `<HelperText>` với tin nhắn rỗng hoặc undefined — nó vẫn sẽ chiếm không gian với margin `mt-[4px]`. Render có điều kiện thay vào đó.

**Sai:**

```tsx
// ❌ HelperText rỗng vẫn thêm margin top
<HelperText color="error">{errorMessage}</HelperText>
```

**Đúng:**

```tsx
// ✅ Chỉ render khi có tin nhắn
{
  errorMessage && <HelperText color="error">{errorMessage}</HelperText>;
}
```
