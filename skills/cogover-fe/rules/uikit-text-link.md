---
title: Cách sử dụng TextLink Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng TextLink sai dẫn đến việc truyền icon sai vị trí, không tận dụng được prop component để tích hợp với router, và chọn size không phù hợp
tags: text-link, link, anchor, navigation, icon, ui-kit
---

## Cách sử dụng TextLink Component của ui-kit

Đường dẫn component: `ui-kit/src/components/TextLink`

Props: `component`, `size`, `underline`, `startIcon`, `endIcon`, `href`, `to`, `target`, `className`. Hỗ trợ `ref` forwarding tới `HTMLAnchorElement`.

---

### RULE-TEXTLINK-01: Sử dụng prop `component` để tích hợp với router, không wrap TextLink bằng Link

Khi cần điều hướng bằng router (ví dụ: React Router), truyền component Link qua prop `component` và sử dụng prop `to`. KHÔNG wrap `TextLink` bên trong `Link` hoặc ngược lại.

**Sai:**

```tsx
// ❌ Wrap thủ công — mất semantic và tạo nested anchor
<Link to="/settings">
  <TextLink>Cài đặt</TextLink>
</Link>
```

**Đúng:**

```tsx
// ✅ Truyền Link component qua prop component
<TextLink component={Link} to="/settings">Cài đặt</TextLink>
```

---

### RULE-TEXTLINK-02: Truyền icons qua `startIcon`/`endIcon`, không đặt trong children

**Sai:**

```tsx
// ❌ Icon lẫn trong children — mất khoảng cách chuẩn và layout nhất quán
<TextLink href="/docs">
  <ExternalLinkIcon /> Tài liệu
</TextLink>
```

**Đúng:**

```tsx
// ✅ Sử dụng startIcon hoặc endIcon
<TextLink href="/docs" endIcon={<ExternalLinkIcon />}>Tài liệu</TextLink>
<TextLink href="/back" startIcon={<ArrowLeftIcon />}>Quay lại</TextLink>
```

---

### RULE-TEXTLINK-03: Sử dụng `size` phù hợp với ngữ cảnh, không override font-size thủ công

TextLink hỗ trợ 3 kích thước: `"small"`, `"medium"` (mặc định), và `"large"`. Sử dụng prop `size` thay vì tự thêm class font-size.

**Sai:**

```tsx
// ❌ Override font-size thủ công — không đồng bộ với design system
<TextLink className={cx("text-[0.75rem]")} href="/terms">
  Điều khoản
</TextLink>
```

**Đúng:**

```tsx
// ✅ Sử dụng prop size
<TextLink size="small" href="/terms">Điều khoản</TextLink>
<TextLink size="large" href="/docs">Tài liệu hướng dẫn</TextLink>

// ✅ Bỏ qua size khi dùng giá trị mặc định (medium)
<TextLink href="/help">Trợ giúp</TextLink>
```

---

### RULE-TEXTLINK-04: Không truyền rõ ràng giá trị mặc định cho `size`, `underline` và `component`

`size` mặc định là `"medium"`, `underline` mặc định là `false`, và `component` mặc định là `"a"`. Chỉ truyền khi cần thay đổi giá trị mặc định.

**Sai:**

```tsx
// ❌ Thừa — tất cả đều là giá trị mặc định
<TextLink component="a" size="medium" underline={false} href="/about">
  Giới thiệu
</TextLink>
```

**Đúng:**

```tsx
// ✅ Bỏ qua khi dùng giá trị mặc định
<TextLink href="/about">Giới thiệu</TextLink>

// ✅ Chỉ truyền khi cần thay đổi
<TextLink underline href="/terms">Điều khoản sử dụng</TextLink>
```

---

### RULE-TEXTLINK-05: Sử dụng `underline` khi cần nhấn mạnh tính tương tác của link

Prop `underline` thêm hiệu ứng gạch chân khi hover. Sử dụng khi link nằm trong đoạn văn bản dài hoặc cần phân biệt rõ với text thường.

**Sai:**

```tsx
// ❌ Thêm class underline thủ công — không đúng behavior hover
<TextLink className={cx("underline")} href="/policy">
  Chính sách bảo mật
</TextLink>
```

**Đúng:**

```tsx
// ✅ Sử dụng prop underline — tự động thêm hover:underline
<TextLink underline href="/policy">Chính sách bảo mật</TextLink>
```

---

### RULE-TEXTLINK-06: Luôn truyền `target="_blank"` khi link mở trang ngoài

Khi link trỏ đến domain bên ngoài, nên truyền `target="_blank"` để mở tab mới, tránh người dùng rời khỏi ứng dụng.

**Sai:**

```tsx
// ❌ Link ngoài nhưng không mở tab mới — người dùng bị chuyển khỏi ứng dụng
<TextLink href="https://docs.example.com">Tài liệu API</TextLink>
```

**Đúng:**

```tsx
// ✅ Mở tab mới cho link ngoài
<TextLink href="https://docs.example.com" target="_blank">
  Tài liệu API
</TextLink>
```

---

## Tham chiếu kích thước

| size | Typography class |
|------|-----------------|
| `small` | `prose-text-link-s` |
| `medium` | `prose-text-link-m` |
| `large` | `prose-subtitle1` |
