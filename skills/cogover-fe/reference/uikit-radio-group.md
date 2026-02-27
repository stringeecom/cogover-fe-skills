---
title: Cách sử dụng component Radio và RadioGroup trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến trạng thái checked không đồng bộ, RadioGroup không chọn đúng giá trị, và lỗi accessibility
tags: radio, radio-group, form, input, ui-kit, form-control-label
---

## Cách sử dụng component Radio và RadioGroup trong ui-kit

Đường dẫn component:
- `ui-kit/src/components/Radio`
- `ui-kit/src/components/RadioGroup`
- `ui-kit/src/components/FormControlLabel` (dùng kèm RadioGroup)

**Radio** props: `checked`, `onChange`, `disabled`, `size`, `value`, `className`. Extends `Omit<InputHTMLAttributes, "size" | "defaultChecked" | "value">`.
**RadioGroup** props: `value` (T | null), `onChange` (e, value) => void, `row`, `disabled`, `className`, `name`. Generic type `RadioGroupProps<T>`.
**FormControlLabel** props: `label`, `control`, `labelPlacement`, `labelAlign`, `disabled`, `disabledLabel`, `usingHtmlFor`, `className`, `classNameLabel`.

Giá trị mặc định: Radio `size="medium"`. RadioGroup `row=false`. FormControlLabel `labelPlacement="right"`, `disabledLabel=true`, `usingHtmlFor=true`.

---

### RULE-RADIO-01: RadioGroup luôn là controlled — phải truyền `value` và `onChange`

`RadioGroup` yêu cầu `value` (giá trị hiện tại hoặc `null`) và `onChange(e, value)` để hoạt động. Component không có uncontrolled mode. Không sử dụng state nội bộ của từng Radio con.

**Sai:**

```tsx
// ❌ Thiếu value và onChange — RadioGroup không hoạt động
<RadioGroup>
  <Radio value="a" />
  <Radio value="b" />
</RadioGroup>

// ❌ Tự quản lý checked trên từng Radio con trong RadioGroup
<RadioGroup value={selected} onChange={(e, val) => setSelected(val)}>
  <Radio value="a" checked={selected === "a"} />
  <Radio value="b" checked={selected === "b"} />
</RadioGroup>
```

**Đúng:**

```tsx
// ✅ RadioGroup tự xử lý checked state dựa trên value
<RadioGroup value={selected} onChange={(e, val) => setSelected(val)}>
  <Radio value="a" />
  <Radio value="b" />
</RadioGroup>
```

---

### RULE-RADIO-02: Sử dụng `RadioGroup` để quản lý nhóm radio — không tự viết logic chọn

`RadioGroup` nhận `value` (T | null) và `onChange(e, value)`. Component tự xử lý việc so sánh value và set checked cho từng Radio con. Không tự viết logic so sánh và toggle.

**Sai:**

```tsx
// ❌ Tự quản lý logic chọn cho từng radio
<div className={cx("flex flex-col gap-[0.75rem]")}>
  <Radio
    value="a"
    checked={selected === "a"}
    onChange={() => setSelected("a")}
  />
  <Radio
    value="b"
    checked={selected === "b"}
    onChange={() => setSelected("b")}
  />
</div>
```

**Đúng:**

```tsx
// ✅ RadioGroup tự xử lý checked và onChange
<RadioGroup value={selected} onChange={(e, val) => setSelected(val)}>
  <Radio value="a" />
  <Radio value="b" />
</RadioGroup>
```

---

### RULE-RADIO-03: Sử dụng `FormControlLabel` để thêm label cho radio trong RadioGroup

Khi cần label bên cạnh radio trong `RadioGroup`, wrap bằng `FormControlLabel` với prop `control`. Không tự tạo layout label + radio.

**Sai:**

```tsx
// ❌ Tự tạo layout label + radio
<RadioGroup value={selected} onChange={(e, val) => setSelected(val)}>
  <div className={cx("flex items-center gap-[0.5rem]")}>
    <Radio value="a" />
    <span>Lựa chọn A</span>
  </div>
  <div className={cx("flex items-center gap-[0.5rem]")}>
    <Radio value="b" />
    <span>Lựa chọn B</span>
  </div>
</RadioGroup>
```

**Đúng:**

```tsx
// ✅ FormControlLabel tự xử lý layout, htmlFor, và disabled state
<RadioGroup value={selected} onChange={(e, val) => setSelected(val)}>
  <FormControlLabel label="Lựa chọn A" control={<Radio value="a" />} />
  <FormControlLabel label="Lựa chọn B" control={<Radio value="b" />} />
</RadioGroup>
```

---

### RULE-RADIO-04: Sử dụng `row` trên RadioGroup cho layout ngang — không override flex direction

Mặc định `row=false` (vertical, flex-col). Truyền `row={true}` để chuyển sang layout ngang (flex-row với items-center). Không override flex direction qua className.

**Sai:**

```tsx
// ❌ Override flex direction qua className
<RadioGroup
  className={cx("!flex-row !items-center")}
  value={selected}
  onChange={(e, val) => setSelected(val)}
>
  <Radio value="a" />
  <Radio value="b" />
</RadioGroup>
```

**Đúng:**

```tsx
// ✅
<RadioGroup row value={selected} onChange={(e, val) => setSelected(val)}>
  <Radio value="a" />
  <Radio value="b" />
</RadioGroup>
```

---

### RULE-RADIO-05: `disabled` trên RadioGroup override disabled của từng children

Khi truyền `disabled` trên `RadioGroup`, tất cả children bị disabled bất kể disabled riêng của chúng. Chỉ truyền `disabled` trên từng child khi cần disable một số cụ thể.

**Sai:**

```tsx
// ❌ Truyền disabled trên từng child khi muốn disable tất cả
<RadioGroup value={selected} onChange={(e, val) => setSelected(val)}>
  <Radio value="a" disabled />
  <Radio value="b" disabled />
  <Radio value="c" disabled />
</RadioGroup>
```

**Đúng:**

```tsx
// ✅ Disable toàn bộ group
<RadioGroup disabled value={selected} onChange={(e, val) => setSelected(val)}>
  <Radio value="a" />
  <Radio value="b" />
  <Radio value="c" />
</RadioGroup>

// ✅ Disable một số cụ thể
<RadioGroup value={selected} onChange={(e, val) => setSelected(val)}>
  <Radio value="a" />
  <Radio value="b" disabled />
  <Radio value="c" />
</RadioGroup>
```

---

### RULE-RADIO-06: Mỗi Radio trong RadioGroup phải có `value` unique

`RadioGroup` sử dụng `value` prop của mỗi child Radio làm key và để xác định checked state. Nếu thiếu `value` hoặc trùng `value`, việc chọn sẽ sai.

**Sai:**

```tsx
// ❌ Thiếu value — RadioGroup không thể xác định checked state
<RadioGroup value={selected} onChange={(e, val) => setSelected(val)}>
  <Radio />
  <Radio />
</RadioGroup>

// ❌ Value trùng — chọn một cái sẽ checked cả hai
<RadioGroup value={selected} onChange={(e, val) => setSelected(val)}>
  <Radio value="same" />
  <Radio value="same" />
</RadioGroup>
```

**Đúng:**

```tsx
// ✅ Mỗi radio có value unique
<RadioGroup value={selected} onChange={(e, val) => setSelected(val)}>
  <Radio value="option1" />
  <Radio value="option2" />
  <Radio value="option3" />
</RadioGroup>
```

---

### RULE-RADIO-07: Sử dụng `size` trên Radio cho kích thước — không override width/height qua className

Ba kích thước có sẵn: `"large"` (24px), `"medium"` (20px, mặc định), `"small"` (16px). Không tự override kích thước qua className.

**Sai:**

```tsx
// ❌ Override kích thước qua className — phá vỡ border-radius và icon scaling
<Radio checked={value} onChange={handleChange} className={cx("w-[2rem] h-[2rem]")} />
```

**Đúng:**

```tsx
// ✅
<Radio checked={value} onChange={handleChange} size="large" />
<Radio checked={value} onChange={handleChange} size="small" />
```

---

### RULE-RADIO-08: Không truyền `type` — component luôn là radio

Input type luôn được set cứng là `"radio"` bên trong component. Không truyền `type` prop.

**Sai:**

```tsx
// ❌ Thừa — type luôn là "radio"
<Radio type="radio" checked={value} onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅
<Radio checked={value} onChange={handleChange} />
```

---

### RULE-RADIO-09: Sử dụng `labelPlacement` trên FormControlLabel để đổi vị trí label — không đảo thứ tự JSX

`FormControlLabel` hỗ trợ `labelPlacement`: `"right"` (mặc định), `"left"`, `"top"`, `"bottom"`. Không đảo thứ tự control và label trong JSX.

**Sai:**

```tsx
// ❌ Đảo thứ tự JSX để label ở bên trái
<div className={cx("flex items-center gap-[0.5rem]")}>
  <span>Label</span>
  <Radio checked={value} onChange={handleChange} />
</div>
```

**Đúng:**

```tsx
// ✅ Label bên trái
<FormControlLabel
  label="Label"
  control={<Radio checked={value} onChange={handleChange} />}
  labelPlacement="left"
/>

// ✅ Label phía trên
<FormControlLabel
  label="Label"
  control={<Radio checked={value} onChange={handleChange} />}
  labelPlacement="top"
/>
```

---

### RULE-RADIO-10: `onChange` của RadioGroup trả về giá trị của radio được chọn — không lấy từ event

Callback `onChange` của `RadioGroup` có signature `(e: ChangeEvent<HTMLInputElement>, value: T)`. Tham số `value` thứ hai chính là giá trị của radio được chọn. Không cần trích xuất giá trị từ `e.target.value`.

**Sai:**

```tsx
// ❌ Lấy value từ event — có thể bị ép kiểu thành string
<RadioGroup
  value={selected}
  onChange={(e) => setSelected(e.target.value)}
>
  <Radio value={1} />
  <Radio value={2} />
</RadioGroup>
```

**Đúng:**

```tsx
// ✅ Dùng tham số value thứ hai — giữ nguyên kiểu dữ liệu gốc (number, string, boolean)
<RadioGroup
  value={selected}
  onChange={(e, val) => setSelected(val)}
>
  <Radio value={1} />
  <Radio value={2} />
</RadioGroup>
```
