---
title: Cách sử dụng UrlInput Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng UrlInput sai dẫn đến việc nút mở URL không hoạt động, display text không tự động điền, và duplicate logic với TextField
tags: url, input, link, display-text, popper, text-field, ui-kit
---

## Cách sử dụng UrlInput Component của ui-kit

Đường dẫn component: `ui-kit/src/components/UrlInput`

UrlInput là wrapper của `TextField` với nút mở URL (target blank) được tích hợp sẵn bên phải. Hỗ trợ tính năng display text qua Popper popup và tự động điền display text từ title của URL. Props kế thừa toàn bộ từ `TextFieldProps`: `label`, `error`, `helperText`, `fullWidth`, `startAdornment`, `endAdornment`, `disabled`, `readOnly`, `value`, `onChange`, `placeholder`, `className`, `inputClassName`, và tất cả `React.InputHTMLAttributes`. Hỗ trợ `ref` forwarding.

Props riêng của UrlInput: `hasDisplayText`, `displayText`, `defaultDisplayText`, `onChangeDisplayText`, `displayTextPlaceholder`, `autoFillDisplayText`, `wrapperClassName`, `popperProps`, `displayTextInputProps`.

---

### RULE-URL-01: Sử dụng `UrlInput` thay vì tự tạo TextField + nút mở link

Khi cần input URL có nút mở link trong tab mới, sử dụng `UrlInput`. Không tự ghép `TextField` với nút mở link thủ công.

**Sai:**

```tsx
// ❌ Tự tạo layout TextField + nút mở link — UrlInput đã làm điều này
<div className={cx("inline-flex")}>
  <TextField value={url} onChange={handleChange} />
  <a href={url} target="_blank">
    <FontAwesomeIcon icon={faGlobe} />
  </a>
</div>
```

**Đúng:**

```tsx
// ✅ UrlInput đã tích hợp sẵn nút mở URL
<UrlInput value={url} onChange={handleChange} />
```

---

### RULE-URL-02: Luôn truyền `value` để nút mở URL hoạt động đúng

Nút mở URL sử dụng `value` để tạo link `href`. Nếu không truyền `value`, nút sẽ bị vô hiệu hóa (pointer-events-none) và link sẽ không có địa chỉ URL.

**Sai:**

```tsx
// ❌ Không truyền value — nút mở URL luôn bị vô hiệu hóa
<UrlInput onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅ Truyền value để nút mở URL hoạt động khi có giá trị
<UrlInput value={url} onChange={handleChange} />
```

---

### RULE-URL-03: Sử dụng `hasDisplayText` khi cần hiển thị text thay thế cho URL

Khi `hasDisplayText={true}`, UrlInput sẽ hiển thị Popper popup chứa input display text khi focus. Khi không focus, input sẽ hiển thị display text thay vì URL. Mặc định `hasDisplayText` là `false`.

**Sai:**

```tsx
// ❌ Tự tạo Popper riêng cho display text — UrlInput đã hỗ trợ sẵn
<UrlInput value={url} onChange={handleChange} />
<Popper>
  <TextField value={displayText} onChange={handleChangeDisplayText} />
</Popper>
```

**Đúng:**

```tsx
// ✅ Sử dụng hasDisplayText để bật tính năng display text
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

### RULE-URL-04: Truyền `autoFillDisplayText={false}` khi không muốn tự động lấy title từ URL

Mặc định `autoFillDisplayText` là `true`. Khi bật, component sẽ tự động gọi API lấy title của URL để điền vào display text (với debounce 1 giây). Tắt tính năng này nếu không cần hoặc nếu muốn người dùng tự nhập display text.

**Đúng:**

```tsx
// ✅ Tắt auto fill khi muốn người dùng tự nhập display text
<UrlInput
  value={url}
  onChange={handleChange}
  hasDisplayText
  displayText={displayText}
  onChangeDisplayText={handleChangeDisplayText}
  autoFillDisplayText={false}
/>

// ✅ Mặc định autoFillDisplayText là true — tự động lấy title từ URL
<UrlInput
  value={url}
  onChange={handleChange}
  hasDisplayText
  displayText={displayText}
  onChangeDisplayText={handleChangeDisplayText}
/>
```

---

### RULE-URL-05: Sử dụng `fullWidth` khi cần UrlInput chiếm toàn bộ chiều rộng

Mặc định UrlInput là `inline-flex`. Truyền `fullWidth` để UrlInput chiếm toàn bộ chiều rộng container cha.

**Sai:**

```tsx
// ❌ Dùng className để override chiều rộng
<UrlInput className={cx("w-full")} value={url} onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅ Sử dụng prop fullWidth
<UrlInput fullWidth value={url} onChange={handleChange} />
```

---

### RULE-URL-06: Sử dụng `wrapperClassName` để style container ngoài, không dùng `className`

Prop `className` sẽ được truyền xuống `TextField` bên trong. Để style container wrapper bên ngoài, sử dụng `wrapperClassName`.

**Sai:**

```tsx
// ❌ className được áp dụng cho TextField, không phải wrapper
<UrlInput className={cx("mb-[1rem]")} value={url} onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅ Sử dụng wrapperClassName cho container ngoài
<UrlInput wrapperClassName={cx("mb-[1rem]")} value={url} onChange={handleChange} />
```

---

### RULE-URL-07: Sử dụng `displayTextInputProps` để tùy chỉnh input display text trong Popper

Khi cần tùy chỉnh sự kiện hoặc props của input display text bên trong Popper, sử dụng `displayTextInputProps` thay vì cố gắng truy cập trực tiếp.

**Đúng:**

```tsx
// ✅ Sử dụng displayTextInputProps để thêm sự kiện cho input display text
<UrlInput
  value={url}
  onChange={handleChange}
  hasDisplayText
  displayText={displayText}
  onChangeDisplayText={handleChangeDisplayText}
  displayTextInputProps={{
    onKeyDown: handleKeyDown,
    onChange: handleDisplayTextInputChange,
  }}
/>
```

---

### RULE-URL-08: Popper display text tự động bị vô hiệu hóa khi `disabled` hoặc `readOnly`

Khi UrlInput có prop `disabled` hoặc `readOnly`, Popper display text sẽ không hiển thị dù `hasDisplayText` là `true`. Không cần tự xử lý logic này.

**Sai:**

```tsx
// ❌ Tự quản lý việc ẩn display text khi disabled — UrlInput đã xử lý
<UrlInput
  value={url}
  onChange={handleChange}
  hasDisplayText={!isDisabled}
  disabled={isDisabled}
/>
```

**Đúng:**

```tsx
// ✅ Popper tự động bị vô hiệu hóa khi disabled
<UrlInput
  value={url}
  onChange={handleChange}
  hasDisplayText
  disabled={isDisabled}
/>
```

---

## Tham chiếu cấu trúc

| Phần tử | Mô tả |
|---------|-------|
| Container ngoài | `inline-flex`, thêm `w-full` khi `fullWidth`, style qua `wrapperClassName` |
| TextField (bên trái) | Chiếm `w-full` bên trong Popper trigger, border-radius phải bị loại bỏ (`rounded-r-none`) |
| Nút mở URL (bên phải) | `40x40px`, `rounded-r`, màu nền `bg-primary-light-90`, bị vô hiệu hóa khi `value` rỗng |
| Popper popup | Hiển thị khi focus và `hasDisplayText={true}`, chứa FormItem với TextField cho display text |

| Trạng thái nút mở URL | Màu nền | Màu icon |
|----------------------|---------|----------|
| Mặc định | `bg-primary-light-90` | `text-typo-secondary` |
| Hover | `bg-primary-light-80` | `text-primary-main` |
| Active | `bg-primary-light-90` | `text-primary-light-10` |
| Vô hiệu hóa (value rỗng) | `pointer-events-none` | -- |
