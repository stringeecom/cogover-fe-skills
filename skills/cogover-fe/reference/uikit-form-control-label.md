---
title: Cách sử dụng component FormControlLabel trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến label không liên kết đúng với control, disabled state không đồng bộ, layout bị lệch, và lỗi accessibility
tags: form-control-label, label, form, checkbox, radio, switch, ui-kit
---

## Cách sử dụng component FormControlLabel trong ui-kit

Đường dẫn component: `ui-kit/src/components/FormControlLabel`

**FormControlLabel** props: `label` (ReactNode), `control` (ReactNode), `labelPlacement`, `labelAlign`, `disabled`, `disabledLabel`, `usingHtmlFor`, `className`, `classNameLabel`, `name`. Hỗ trợ spread thêm props vào container div.

Giá trị mặc định: `labelPlacement="right"`, `disabledLabel=true`, `usingHtmlFor=true`.

Kiểu dữ liệu:
- `labelPlacement`: `"top" | "left" | "right" | "bottom"`
- `labelAlign`: `"left" | "right" | "center"`

---

### RULE-FCL-01: Sử dụng `FormControlLabel` để gắn label cho control — không tự tạo layout label + control

`FormControlLabel` tự động xử lý layout, `htmlFor`, disabled state và cursor. Không tự tạo layout bằng div + span.

**Sai:**

```tsx
// ❌ Tự tạo layout label + control — thiếu htmlFor, thiếu disabled sync
<div className={cx("flex items-center gap-[0.5rem]")}>
  <Checkbox checked={value} onChange={handleChange} />
  <span>Đồng ý điều khoản</span>
</div>
```

**Đúng:**

```tsx
// ✅ FormControlLabel tự xử lý layout, htmlFor, disabled state
<FormControlLabel
  label="Đồng ý điều khoản"
  control={<Checkbox checked={value} onChange={handleChange} />}
/>
```

---

### RULE-FCL-02: Truyền component vào prop `control` — không truyền vào children

`FormControlLabel` sử dụng prop `control` để nhận component (Checkbox, Radio, Switch, ...). Component không hỗ trợ children.

**Sai:**

```tsx
// ❌ Truyền control qua children — không hoạt động
<FormControlLabel label="Tùy chọn A">
  <Checkbox value="a" />
</FormControlLabel>
```

**Đúng:**

```tsx
// ✅ Truyền qua prop control
<FormControlLabel label="Tùy chọn A" control={<Checkbox value="a" />} />
```

---

### RULE-FCL-03: Sử dụng `labelPlacement` để đổi vị trí label — không đảo thứ tự JSX hoặc override flex direction

`FormControlLabel` hỗ trợ `labelPlacement`: `"right"` (mặc định), `"left"`, `"top"`, `"bottom"`. Component tự xử lý flex direction tương ứng.

**Sai:**

```tsx
// ❌ Override flex direction qua className
<FormControlLabel
  label="Label"
  control={<Checkbox checked={value} onChange={handleChange} />}
  className={cx("!flex-row-reverse")}
/>

// ❌ Tự tạo layout để đặt label bên trái
<div className={cx("flex items-center gap-[0.5rem]")}>
  <span>Label</span>
  <Checkbox checked={value} onChange={handleChange} />
</div>
```

**Đúng:**

```tsx
// ✅ Label bên trái
<FormControlLabel
  label="Label"
  control={<Checkbox checked={value} onChange={handleChange} />}
  labelPlacement="left"
/>

// ✅ Label phía trên
<FormControlLabel
  label="Label"
  control={<Checkbox checked={value} onChange={handleChange} />}
  labelPlacement="top"
/>

// ✅ Label phía dưới
<FormControlLabel
  label="Label"
  control={<Checkbox checked={value} onChange={handleChange} />}
  labelPlacement="bottom"
/>
```

---

### RULE-FCL-04: Sử dụng `labelAlign` để căn chỉnh label — không override alignment qua className

`labelAlign` hoạt động khác nhau tùy theo `labelPlacement`:
- Khi `labelPlacement` là `"left"` hoặc `"right"` (ngang): `labelAlign` căn chỉnh nội dung label theo `justify-start`, `justify-center`, `justify-end` với width cố định `140px`.
- Khi `labelPlacement` là `"top"` hoặc `"bottom"` (dọc): `labelAlign` căn chỉnh items theo `items-start`, `items-center`, `items-end`.

**Sai:**

```tsx
// ❌ Override alignment qua className
<FormControlLabel
  label="Label"
  control={<Checkbox checked={value} onChange={handleChange} />}
  classNameLabel={cx("!text-right")}
/>
```

**Đúng:**

```tsx
// ✅ Căn phải khi layout ngang
<FormControlLabel
  label="Label"
  control={<Checkbox checked={value} onChange={handleChange} />}
  labelPlacement="left"
  labelAlign="right"
/>

// ✅ Căn trái khi layout dọc
<FormControlLabel
  label="Label"
  control={<Checkbox checked={value} onChange={handleChange} />}
  labelPlacement="top"
  labelAlign="left"
/>
```

---

### RULE-FCL-05: Truyền `disabled` trên FormControlLabel để đồng bộ disabled state cho cả control và label

Khi truyền `disabled` trên `FormControlLabel`, component tự đồng bộ disabled cho control bên trong và thay đổi style label (màu `text-divider-primary`, bỏ `cursor-pointer`). Nếu không truyền `disabled` trên FormControlLabel, component sẽ lấy disabled từ props của control.

**Sai:**

```tsx
// ❌ Chỉ disable control nhưng label vẫn có cursor pointer và màu bình thường
<FormControlLabel
  label="Tùy chọn"
  control={<Checkbox checked={value} onChange={handleChange} disabled />}
/>
// Lưu ý: trường hợp này vẫn hoạt động vì component lấy disabled từ control,
// nhưng nên ưu tiên truyền disabled trên FormControlLabel cho rõ ràng
```

**Đúng:**

```tsx
// ✅ Disabled đồng bộ cho cả control và label
<FormControlLabel
  label="Tùy chọn"
  control={<Checkbox checked={value} onChange={handleChange} />}
  disabled
/>
```

---

### RULE-FCL-06: Sử dụng `disabledLabel` để kiểm soát style label khi disabled — mặc định là `true`

Khi `disabledLabel=true` (mặc định), label sẽ đổi sang màu `text-divider-primary` khi disabled. Truyền `disabledLabel={false}` nếu muốn label giữ nguyên màu dù control bị disabled.

**Sai:**

```tsx
// ❌ Override màu label qua classNameLabel khi disabled
<FormControlLabel
  label="Tùy chọn"
  control={<Checkbox checked={value} onChange={handleChange} />}
  disabled
  classNameLabel={cx("!text-typo-primary")}
/>
```

**Đúng:**

```tsx
// ✅ Giữ nguyên màu label khi disabled
<FormControlLabel
  label="Tùy chọn"
  control={<Checkbox checked={value} onChange={handleChange} />}
  disabled
  disabledLabel={false}
/>
```

---

### RULE-FCL-07: Sử dụng `usingHtmlFor={false}` khi control không hỗ trợ htmlFor — mặc định là `true`

Mặc định `usingHtmlFor=true`, label sẽ có thuộc tính `htmlFor` liên kết với id của control. Khi control không phải input element (ví dụ: custom component không có id), truyền `usingHtmlFor={false}` để tránh htmlFor trỏ sai.

**Sai:**

```tsx
// ❌ usingHtmlFor mặc định true nhưng control không có id — htmlFor trỏ đến id tự sinh không match
<FormControlLabel
  label="Tùy chọn"
  control={<CustomToggle onToggle={handleToggle} />}
/>
```

**Đúng:**

```tsx
// ✅ Tắt htmlFor khi control không phải input element
<FormControlLabel
  label="Tùy chọn"
  control={<CustomToggle onToggle={handleToggle} />}
  usingHtmlFor={false}
/>

// ✅ Giữ usingHtmlFor mặc định cho các control chuẩn (Checkbox, Radio, Switch)
<FormControlLabel
  label="Tùy chọn"
  control={<Checkbox checked={value} onChange={handleChange} />}
/>
```

---

### RULE-FCL-08: Sử dụng `classNameLabel` để style label — không override label style qua `className`

`className` áp dụng cho container div bên ngoài. `classNameLabel` áp dụng trực tiếp cho thẻ `<label>`. Không dùng `className` với selector con để style label.

**Sai:**

```tsx
// ❌ Dùng className với selector con để style label
<FormControlLabel
  label="Tùy chọn"
  control={<Checkbox checked={value} onChange={handleChange} />}
  className={cx("[&>label]:text-typo-secondary [&>label]:prose-body1")}
/>
```

**Đúng:**

```tsx
// ✅ Dùng classNameLabel cho label
<FormControlLabel
  label="Tùy chọn"
  control={<Checkbox checked={value} onChange={handleChange} />}
  classNameLabel={cx("text-typo-secondary prose-body1")}
/>

// ✅ Dùng className cho container
<FormControlLabel
  label="Tùy chọn"
  control={<Checkbox checked={value} onChange={handleChange} />}
  className={cx("mb-[1rem]")}
/>
```

---

### RULE-FCL-09: Label hỗ trợ ReactNode — sử dụng cho nội dung phức tạp

Prop `label` có kiểu `React.ReactNode`, hỗ trợ string, JSX, hoặc component. Khi label rỗng (undefined, null, empty string), label sẽ không được render.

**Sai:**

```tsx
// ❌ Tự wrap thêm một lớp ngoài label
<FormControlLabel
  label={
    <div className={cx("flex items-center gap-[0.25rem]")}>
      <span>Tùy chọn</span>
    </div>
  }
  control={<Checkbox checked={value} onChange={handleChange} />}
/>
```

**Đúng:**

```tsx
// ✅ Truyền string đơn giản
<FormControlLabel
  label="Tùy chọn"
  control={<Checkbox checked={value} onChange={handleChange} />}
/>

// ✅ Truyền JSX phức tạp khi cần
<FormControlLabel
  label={
    <span className={cx("flex items-center gap-[0.25rem]")}>
      Đồng ý với <a href="/terms" className={cx("text-primary underline")}>điều khoản</a>
    </span>
  }
  control={<Checkbox checked={value} onChange={handleChange} />}
/>
```
