## TextLink Component (`uikit-text-link`)

Đường dẫn: `ui-kit/src/components/TextLink`

Props: `component` (mặc định `"a"`), `size` ("small"|"medium"|"large", mặc định `"medium"`), `underline` (mặc định `false`), `startIcon`, `endIcon`, `href`, `to`, `target`, `className`. Hỗ trợ `ref` forwarding tới `HTMLAnchorElement`.

| size | Typography class |
|------|-----------------|
| `small` | `prose-text-link-s` |
| `medium` | `prose-text-link-m` |
| `large` | `prose-subtitle1` |

---

### RULE-TEXTLINK-01: Dùng prop `component` để tích hợp với router — không wrap TextLink bằng Link

```tsx
// ❌ Nested anchor — mất semantic
<Link to="/settings"><TextLink>Cài đặt</TextLink></Link>

// ✅
<TextLink component={Link} to="/settings">Cài đặt</TextLink>
```

---

### RULE-TEXTLINK-02: Truyền icons qua `startIcon`/`endIcon` — không đặt trong children

```tsx
// ❌ Mất khoảng cách chuẩn và layout nhất quán
<TextLink href="/docs"><ExternalLinkIcon /> Tài liệu</TextLink>

// ✅
<TextLink href="/docs" endIcon={<ExternalLinkIcon />}>Tài liệu</TextLink>
<TextLink href="/back" startIcon={<ArrowLeftIcon />}>Quay lại</TextLink>
```

---

### RULE-TEXTLINK-03: Dùng prop `size` — không override font-size thủ công

```tsx
// ❌
<TextLink className={cx("text-[0.75rem]")} href="/terms">Điều khoản</TextLink>

// ✅
<TextLink size="small" href="/terms">Điều khoản</TextLink>
<TextLink href="/help">Trợ giúp</TextLink>  // medium là mặc định
```

---

### RULE-TEXTLINK-04: Không truyền rõ ràng giá trị mặc định

```tsx
// ❌ Thừa
<TextLink component="a" size="medium" underline={false} href="/about">Giới thiệu</TextLink>

// ✅
<TextLink href="/about">Giới thiệu</TextLink>
<TextLink underline href="/terms">Điều khoản sử dụng</TextLink>
```

---

### RULE-TEXTLINK-05: Dùng prop `underline` — không thêm class underline thủ công

```tsx
// ❌ Không đúng behavior hover
<TextLink className={cx("underline")} href="/policy">Chính sách bảo mật</TextLink>

// ✅ Tự động thêm hover:underline
<TextLink underline href="/policy">Chính sách bảo mật</TextLink>
```

---

### RULE-TEXTLINK-06: Luôn truyền `target="_blank"` khi link mở trang ngoài

```tsx
// ❌ Người dùng bị chuyển khỏi ứng dụng
<TextLink href="https://docs.example.com">Tài liệu API</TextLink>

// ✅
<TextLink href="https://docs.example.com" target="_blank">Tài liệu API</TextLink>
```
