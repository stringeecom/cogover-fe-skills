---
title: Cách sử dụng TextField Component của ui-kit
impact: HIGH
impactDescription: Sử dụng TextField sai dẫn đến việc xử lý state không nhất quán, hiển thị label/error không đúng, và tuỳ chỉnh style sai chỗ
tags: text-field, input, textarea, form, adornment, ui-kit
---

## Cách sử dụng TextField Component của ui-kit

Đường dẫn component: `ui-kit/src/components/TextField`

Props: `label`, `error`, `helperText`, `color`, `fullWidth`, `startAdornment`, `endAdornment`, `inputClassName`, `multipleLine`, `rows`, `readOnly`, `htmlReadOnly`, `isFocused`, `hideValue`, `defaultExpand`, `resize`, `hideMaxLength`, `inputStyle`, `enabledOverflowY`, `aiLoading`, `showBorderOnBlur`, `helperTextProps`. Hỗ trợ tất cả `React.InputHTMLAttributes` và `ref` forwarding.

---

### RULE-TF-01: Sử dụng `multipleLine` để chuyển sang textarea, không tự render `<textarea>`

Khi cần nhập nhiều dòng, sử dụng prop `multipleLine={true}` để TextField tự động render `<textarea>`. KHÔNG tự render `<textarea>` riêng hoặc sử dụng component khác.

**Sai:**

```tsx
// ❌ Tự render textarea riêng — mất tất cả tính năng của TextField
<textarea className="custom-textarea" rows={4} />
```

**Đúng:**

```tsx
// ✅ Sử dụng multipleLine để TextField tự xử lý
<TextField multipleLine rows={4} fullWidth />

// ✅ Kết hợp với maxLength để hiển thị bộ đếm ký tự
<TextField multipleLine rows={6} maxLength={500} fullWidth />
```

---

### RULE-TF-02: Truyền icons/elements qua `startAdornment`/`endAdornment`, không đặt trong children hoặc wrapper ngoài

`startAdornment` và `endAdornment` hỗ trợ cả `ReactNode` và `() => ReactNode`. Component tự động tính toán padding cho input dựa trên kích thước của adornment.

**Sai:**

```tsx
// ❌ Wrap icon bên ngoài — mất tính năng tự động tính padding
<div className="relative">
  <SearchIcon className="absolute left-2" />
  <TextField />
</div>
```

**Đúng:**

```tsx
// ✅ Truyền qua startAdornment/endAdornment
<TextField startAdornment={<SearchIcon />} />
<TextField endAdornment={<ClearIcon />} />

// ✅ Hỗ trợ cả function để render có điều kiện
<TextField endAdornment={() => isLoading ? <Spinner /> : <CheckIcon />} />
```

---

### RULE-TF-03: Sử dụng `readOnly` để hiển thị chế độ chỉ đọc, không tự style disabled

Khi cần hiển thị dữ liệu chỉ đọc (không cho chỉnh sửa), sử dụng prop `readOnly={true}`. TextField sẽ render một `<div>` với nội dung đã encode HTML thay vì input. KHÔNG sử dụng `disabled` để thay thế `readOnly` vì disabled thay đổi visual style (nền xám) và không phù hợp với mục đích hiển thị dữ liệu.

**Sai:**

```tsx
// ❌ disabled làm nền xám — không phù hợp để hiển thị dữ liệu
<TextField disabled value="Nguyễn Văn A" />

// ❌ Tự style để hiển thị text — mất tính năng encode HTML và style nhất quán
<div className="text-field-readonly">{value}</div>
```

**Đúng:**

```tsx
// ✅ readOnly render div với nội dung đã encode HTML
<TextField readOnly value="Nguyễn Văn A" fullWidth />
```

---

### RULE-TF-04: Sử dụng `htmlReadOnly` khi cần input không cho nhập nhưng vẫn giữ giao diện input

`htmlReadOnly` chỉ set thuộc tính `readOnly` của thẻ HTML input, giữ nguyên giao diện input (có border, background). Khác với `readOnly` (render thành div).

**Sai:**

```tsx
// ❌ readOnly sẽ render thành div, mất giao diện input
<TextField readOnly value={selectedDate} endAdornment={<CalendarIcon />} />
```

**Đúng:**

```tsx
// ✅ htmlReadOnly giữ giao diện input nhưng không cho nhập
<TextField htmlReadOnly value={selectedDate} endAdornment={<CalendarIcon />} />
```

---

### RULE-TF-05: Sử dụng `error` kết hợp `helperText` để hiển thị lỗi, không tự style border hoặc text lỗi

Khi cần hiển thị trạng thái lỗi, sử dụng `error={true}` kết hợp `helperText` để hiển thị thông báo lỗi. TextField tự động đổi border sang màu `border-error`. KHÔNG tự thêm class border-red hoặc render text lỗi riêng.

**Sai:**

```tsx
// ❌ Tự style border lỗi và render text lỗi riêng
<div>
  <TextField className="border-red-500" />
  <span className="text-red-500">Email không hợp lệ</span>
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng error và helperText
<TextField
  error={!isValidEmail}
  helperText={!isValidEmail ? "Email không hợp lệ" : undefined}
  fullWidth
/>
```

---

### RULE-TF-06: Sử dụng `inputClassName` để tuỳ chỉnh style input, không override bằng `className`

`className` áp dụng cho container ngoài cùng. Khi cần tuỳ chỉnh style của chính thẻ input/textarea, sử dụng `inputClassName`.

**Sai:**

```tsx
// ❌ Override style input qua className của container
<TextField className="[&_input]:text-right" />
```

**Đúng:**

```tsx
// ✅ Sử dụng inputClassName để tuỳ chỉnh style input
<TextField inputClassName={cx("text-right")} />

// ✅ Sử dụng className cho container (ví dụ: thêm margin)
<TextField className={cx("mt-[1rem]")} fullWidth />
```

---

### RULE-TF-07: Sử dụng `hideValue` thay vì tự set `type="password"` để ẩn giá trị

Khi cần ẩn giá trị hiển thị (thay bằng dấu *) nhưng KHÔNG phải trường password (không cần icon toggle show/hide), sử dụng `hideValue={true}`. Prop `type="password"` sẽ tự động hiển thị icon toggle show/hide.

**Sai:**

```tsx
// ❌ type="password" sẽ hiển thị icon toggle — không phù hợp cho trường không phải mật khẩu
<TextField type="password" value={secretKey} />
```

**Đúng:**

```tsx
// ✅ hideValue chỉ ẩn giá trị, không hiển thị icon toggle
<TextField hideValue value={secretKey} />

// ✅ type="password" dành cho trường mật khẩu — có icon toggle show/hide
<TextField type="password" label="Mật khẩu" />
```

---

### RULE-TF-08: Sử dụng `defaultExpand` để textarea tự động giãn theo nội dung

Khi cần textarea tự động điều chỉnh chiều cao theo nội dung, sử dụng `defaultExpand={true}`. Kết hợp `enabledOverflowY={true}` nếu muốn hiển thị scrollbar khi nội dung vượt quá.

**Sai:**

```tsx
// ❌ Tự viết logic autoExpand
const textareaRef = useRef(null);
useEffect(() => {
  textareaRef.current.style.height = textareaRef.current.scrollHeight + 'px';
}, [value]);
<textarea ref={textareaRef} />
```

**Đúng:**

```tsx
// ✅ Sử dụng defaultExpand để tự động giãn
<TextField multipleLine defaultExpand rows={3} fullWidth />

// ✅ Kết hợp enabledOverflowY để hiển thị scrollbar
<TextField multipleLine defaultExpand enabledOverflowY rows={3} fullWidth />
```

---

### RULE-TF-09: Không truyền rõ ràng giá trị mặc định cho `showBorderOnBlur`, `rows`, `resize`

`showBorderOnBlur` mặc định là `true`, `rows` mặc định là `4`, `resize` mặc định là `"none"`. Chỉ truyền khi cần thay đổi giá trị mặc định.

**Sai:**

```tsx
// ❌ Thừa — các giá trị đã là mặc định
<TextField showBorderOnBlur={true} />
<TextField multipleLine rows={4} resize="none" />
```

**Đúng:**

```tsx
// ✅ Bỏ qua khi dùng giá trị mặc định
<TextField />
<TextField multipleLine />

// ✅ Chỉ truyền khi cần thay đổi
<TextField showBorderOnBlur={false} />
<TextField multipleLine rows={8} resize="vertical" />
```

---

### RULE-TF-10: Sử dụng `hideMaxLength` để ẩn bộ đếm ký tự, không tự ẩn bằng CSS

Khi sử dụng `multipleLine` kết hợp `maxLength` nhưng không muốn hiển thị bộ đếm ký tự, sử dụng `hideMaxLength={true}`. KHÔNG ẩn bằng CSS.

**Sai:**

```tsx
// ❌ Ẩn bộ đếm bằng CSS — code khó bảo trì
<TextField
  multipleLine
  maxLength={500}
  className="[&_.stringee-input-max-length]:hidden"
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng hideMaxLength
<TextField multipleLine maxLength={500} hideMaxLength fullWidth />
```

---

## Tham chiếu kích thước

| Chế độ | Chiều cao | Ghi chú |
|--------|-----------|---------|
| Input không có label | 40px | Mặc định |
| Input có label | 56px | Khi truyền prop `label` |
| Textarea (`multipleLine`) | Tự động | Dựa trên `rows`, hoặc tự động giãn khi dùng `defaultExpand` |

## Tham chiếu style trạng thái

| Trạng thái | Border | Background |
|------------|--------|------------|
| Bình thường | `border-divider-primary` | `bg-primary-light-98` |
| Hover | `hover:border-primary-light-70` | `hover:bg-primary-light-96` |
| Focus | `border-info` | `bg-primary-light-96` |
| Error | `border-error` | `bg-primary-light-98` |
| Disabled | `border-primary-light-94` | `bg-primary-light-94` |
