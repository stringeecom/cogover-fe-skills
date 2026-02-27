## EmailInput Component (`uikit-email-input`)

Đường dẫn: `ui-kit/src/components/EmailInput`

Wrapper của `TextField` với nút gửi email (mailto) tích hợp sẵn bên phải. Kế thừa toàn bộ `TextFieldProps` và hỗ trợ `ref` forwarding.

| Phần tử | Mô tả |
|---------|-------|
| Container ngoài | `inline-flex`, thêm `w-full` khi `fullWidth` |
| TextField (bên trái) | `flex-1`, `rounded-r-none` |
| Nút mailto (bên phải) | `40x40px`, `rounded-r`, `bg-primary-light-90`, disabled khi `value` rỗng |

---

### RULE-EMAIL-01: Dùng `EmailInput` — không tự tạo TextField + nút mailto

```tsx
// ❌
<div className={cx("inline-flex")}>
  <TextField value={email} onChange={handleChange} />
  <a href={`mailto:${email}`}><FontAwesomeIcon icon={faEnvelope} /></a>
</div>

// ✅
<EmailInput value={email} onChange={handleChange} />
```

---

### RULE-EMAIL-02: Luôn truyền `value` để nút mailto hoạt động

```tsx
// ❌ Nút luôn bị vô hiệu hóa
<EmailInput onChange={handleChange} />

// ✅
<EmailInput value={email} onChange={handleChange} />
```

---

### RULE-EMAIL-03: Dùng `fullWidth` thay vì className để mở rộng width

```tsx
// ❌
<EmailInput className={cx("w-full")} value={email} onChange={handleChange} />

// ✅
<EmailInput fullWidth value={email} onChange={handleChange} />
```

---

### RULE-EMAIL-04: Không truyền `endAdornment` — nút email đã chiếm vị trí bên phải

```tsx
// ❌ Chồng chéo với nút email
<EmailInput value={email} onChange={handleChange} endAdornment={<FontAwesomeIcon icon={faCheck} />} />

// ✅ Chỉ dùng startAdornment nếu cần
<EmailInput value={email} onChange={handleChange} startAdornment={<FontAwesomeIcon icon={faUser} />} />
```

---

### RULE-EMAIL-05: Dùng các props TextField cho validation và trạng thái

```tsx
// ✅
<EmailInput
  fullWidth
  label="Email"
  required
  value={email}
  onChange={handleChange}
  error={!isValidEmail}
  helperText={!isValidEmail ? "Email không hợp lệ" : undefined}
/>
<EmailInput fullWidth value={email} disabled />
<EmailInput fullWidth value={email} readOnly />
```
