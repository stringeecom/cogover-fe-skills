## Cách sử dụng UrlInput Component của ui-kit

Đường dẫn: `ui-kit/src/components/UrlInput`

Wrapper của `TextField` với nút mở URL (target blank) tích hợp sẵn bên phải. Hỗ trợ display text qua Popper popup và tự động điền display text từ title của URL.

Props kế thừa từ `TextFieldProps` + props riêng:

| Prop | Mô tả |
|------|-------|
| `hasDisplayText` | Bật tính năng display text (Popper popup) |
| `displayText` | Giá trị display text (controlled) |
| `defaultDisplayText` | Giá trị display text mặc định |
| `onChangeDisplayText` | Callback khi display text thay đổi |
| `displayTextPlaceholder` | Placeholder cho input display text |
| `autoFillDisplayText` | Tự động lấy title từ URL (mặc định `true`) |
| `wrapperClassName` | className cho container ngoài |
| `popperProps` | Props cho Popper |
| `displayTextInputProps` | Props cho input display text trong Popper |

---

### RULE-URL-01: Dùng `UrlInput` thay vì tự tạo TextField + nút mở link

```tsx
// ❌
<div className={cx("inline-flex")}>
  <TextField value={url} onChange={handleChange} />
  <a href={url} target="_blank"><FontAwesomeIcon icon={faGlobe} /></a>
</div>

// ✅
<UrlInput value={url} onChange={handleChange} />
```

---

### RULE-URL-02: Luôn truyền `value` để nút mở URL hoạt động

```tsx
// ❌ Nút luôn bị vô hiệu hóa
<UrlInput onChange={handleChange} />

// ✅
<UrlInput value={url} onChange={handleChange} />
```

---

### RULE-URL-03: Dùng `hasDisplayText` khi cần text thay thế cho URL

```tsx
// ✅
<UrlInput
  value={url}
  onChange={handleChange}
  hasDisplayText
  displayText={displayText}
  onChangeDisplayText={handleChangeDisplayText}
  displayTextPlaceholder="Nhập text hiển thị"
/>
```

---

### RULE-URL-04: Dùng `fullWidth` thay vì className để mở rộng width

```tsx
// ❌
<UrlInput className={cx("w-full")} value={url} onChange={handleChange} />

// ✅
<UrlInput fullWidth value={url} onChange={handleChange} />
```

---

### RULE-URL-05: Dùng `wrapperClassName` để style container ngoài, không dùng `className`

```tsx
// ❌ className áp dụng cho TextField bên trong, không phải wrapper
<UrlInput className={cx("mb-[1rem]")} value={url} onChange={handleChange} />

// ✅
<UrlInput wrapperClassName={cx("mb-[1rem]")} value={url} onChange={handleChange} />
```

---

### RULE-URL-06: Popper tự động bị vô hiệu hóa khi `disabled` hoặc `readOnly`

```tsx
// ❌ Tự quản lý hasDisplayText khi disabled — không cần thiết
<UrlInput value={url} onChange={handleChange} hasDisplayText={!isDisabled} disabled={isDisabled} />

// ✅
<UrlInput value={url} onChange={handleChange} hasDisplayText disabled={isDisabled} />
```
