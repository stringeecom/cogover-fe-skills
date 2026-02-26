---
title: Cách sử dụng EmailInput Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng EmailInput sai dẫn đến việc nút gửi email không hoạt động, layout bị vỡ khi thiếu fullWidth, và duplicate logic với TextField
tags: email, input, mailto, text-field, ui-kit
---

## Cách sử dụng EmailInput Component của ui-kit

Đường dẫn component: `ui-kit/src/components/EmailInput`

EmailInput là wrapper của `TextField` với nút gửi email (mailto) được tích hợp sẵn bên phải. Props kế thừa toàn bộ từ `TextFieldProps`: `label`, `error`, `helperText`, `fullWidth`, `startAdornment`, `endAdornment`, `disabled`, `readOnly`, `value`, `onChange`, `placeholder`, `className`, `inputClassName`, và tất cả `React.InputHTMLAttributes`. Hỗ trợ `ref` forwarding.

---

### RULE-EMAIL-01: Sử dụng `EmailInput` thay vì tự tạo TextField + nút mailto

Khi cần input email có nút gửi email, sử dụng `EmailInput`. Không tự ghép `TextField` với nút `mailto` thủ công.

**Sai:**

```tsx
// ❌ Tự tạo layout TextField + nút mailto — EmailInput đã làm điều này
<div className={cx("inline-flex")}>
  <TextField value={email} onChange={handleChange} />
  <a href={`mailto:${email}`}>
    <FontAwesomeIcon icon={faEnvelope} />
  </a>
</div>
```

**Đúng:**

```tsx
// ✅ EmailInput đã tích hợp sẵn nút gửi email
<EmailInput value={email} onChange={handleChange} />
```

---

### RULE-EMAIL-02: Luôn truyền `value` để nút mailto hoạt động đúng

Nút gửi email sử dụng `props.value` để tạo link `mailto:`. Nếu không truyền `value`, nút sẽ bị vô hiệu hóa (pointer-events-none) và link mailto sẽ không có địa chỉ email.

**Sai:**

```tsx
// ❌ Không truyền value — nút mailto luôn bị vô hiệu hóa
<EmailInput onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅ Truyền value để nút mailto hoạt động khi có giá trị
<EmailInput value={email} onChange={handleChange} />
```

---

### RULE-EMAIL-03: Sử dụng `fullWidth` khi cần EmailInput chiếm toàn bộ chiều rộng

Mặc định EmailInput là `inline-flex` và TextField bên trong có chiều rộng 190px. Truyền `fullWidth` để EmailInput chiếm toàn bộ chiều rộng container cha.

**Sai:**

```tsx
// ❌ Dùng className để override chiều rộng
<EmailInput className={cx("w-full")} value={email} onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅ Sử dụng prop fullWidth
<EmailInput fullWidth value={email} onChange={handleChange} />
```

---

### RULE-EMAIL-04: Không truyền `endAdornment` vì nút email đã chiếm vị trí bên phải

EmailInput đã có nút gửi email ở bên phải. Nếu truyền thêm `endAdornment`, nó sẽ hiển thị bên trong TextField và có thể gây chồng chéo giao diện với nút email.

**Sai:**

```tsx
// ❌ endAdornment bị chồng chéo với nút email bên phải
<EmailInput
  value={email}
  onChange={handleChange}
  endAdornment={<FontAwesomeIcon icon={faCheck} />}
/>
```

**Đúng:**

```tsx
// ✅ Chỉ sử dụng startAdornment nếu cần thêm icon
<EmailInput
  value={email}
  onChange={handleChange}
  startAdornment={<FontAwesomeIcon icon={faUser} />}
/>

// ✅ Không truyền endAdornment — nút email đã có sẵn
<EmailInput value={email} onChange={handleChange} />
```

---

### RULE-EMAIL-05: Sử dụng các prop của TextField cho validation và trạng thái

EmailInput kế thừa toàn bộ props từ TextField. Sử dụng `error`, `helperText`, `label`, `disabled`, `required` như khi dùng TextField thông thường.

**Đúng:**

```tsx
// ✅ Sử dụng error + helperText cho validation
<EmailInput
  fullWidth
  label="Email"
  required
  value={email}
  onChange={handleChange}
  error={!isValidEmail}
  helperText={!isValidEmail ? "Email không hợp lệ" : undefined}
/>

// ✅ Sử dụng disabled
<EmailInput fullWidth value={email} disabled />

// ✅ Sử dụng readOnly
<EmailInput fullWidth value={email} readOnly />
```

---

## Tham chiếu cấu trúc

| Phần tử | Mô tả |
|---------|-------|
| Container ngoài | `inline-flex`, thêm `w-full` khi `fullWidth` |
| TextField (bên trái) | Chiếm `flex-1`, border-radius phải bị loại bỏ (`rounded-r-none`) |
| Nút mailto (bên phải) | `40x40px`, `rounded-r`, màu nền `bg-primary-light-90`, bị vô hiệu hóa khi `value` rỗng |

| Trạng thái nút mailto | Màu nền | Màu icon |
|----------------------|---------|----------|
| Mặc định | `bg-primary-light-90` | `text-typo-secondary` |
| Hover | `bg-primary-light-80` | `text-primary-main` |
| Active | `bg-primary-light-90` | `text-primary-light-10` |
| Vô hiệu hóa (value rỗng) | `pointer-events-none` | — |
