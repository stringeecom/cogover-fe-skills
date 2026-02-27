---
title: Cách sử dụng component Radio và RadioGroup trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến trạng thái checked không đồng bộ, RadioGroup không chuyển đổi đúng giá trị, và lỗi accessibility
tags: radio, radio-group, form, input, ui-kit, form-control-label
---

## Cách sử dụng component Radio và RadioGroup trong ui-kit

Đường dẫn component:
- `ui-kit/src/components/Radio`
- `ui-kit/src/components/RadioGroup`
- `ui-kit/src/components/FormControlLabel` (dùng kèm RadioGroup)

**Radio** props: `checked`, `onChange`, `disabled`, `size`, `value`, `className`. Extends `Omit<InputHTMLAttributes, "size" | "defaultChecked" | "value">`.
**RadioGroup** props: `value` (T | null), `onChange` (e, value) => void, `row`, `disabled`, `className`.
**FormControlLabel** props: `label`, `control`, `labelPlacement`, `labelAlign`, `disabled`, `disabledLabel`, `usingHtmlFor`, `className`, `classNameLabel`.

Giá trị mặc định: Radio `size="medium"`. RadioGroup `row=false`. FormControlLabel `labelPlacement="right"`, `disabledLabel=true`, `usingHtmlFor=true`.

---

### RULE-RADIO-01: Pair `checked` với `onChange` trong controlled mode — omit `checked` cho uncontrolled

Khi truyền `checked`, component chạy controlled mode — state do bạn quản lý. Khi không truyền `checked`, component tự quản lý state nội bộ. Không truyền `checked` mà thiếu `onChange` (sẽ không cập nhật được).

**Sai:**

```tsx
// ❌ Truyền checked nhưng thiếu onChange — radio bị "đóng băng"
<Radio checked={isSelected} />

// ❌ Dùng defaultChecked — prop này đã bị omit khỏi type
<Radio defaultChecked={true} onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅ Controlled mode
<Radio checked={isSelected} onChange={(e) => setIsSelected(e.target.checked)} />

// ✅ Uncontrolled mode — component tự quản lý state
<Radio onChange={(e) => handleChange(e)} />
```

---

### RULE-RADIO-02: Sử dụng `size` cho kích thước — không override width/height qua className

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

### RULE-RADIO-03: Không truyền `type` — component luôn là radio

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

### RULE-RADIO-04: Sử dụng `RadioGroup` để quản lý nhóm radio — không tự viết logic chuyển đổi

`RadioGroup` nhận `value` (T | null) và `onChange(e, newValue)`. Component tự xử lý việc chuyển đổi giá trị được chọn. Không tự viết logic checked/onChange cho từng Radio.

**Sai:**

```tsx
// ❌ Tự quản lý logic checked cho từng radio
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
<RadioGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Radio value="a" />
  <Radio value="b" />
</RadioGroup>
```

---

### RULE-RADIO-05: Sử dụng `FormControlLabel` để thêm label cho radio trong RadioGroup

Khi cần label bên cạnh radio trong `RadioGroup`, wrap bằng `FormControlLabel` với prop `control`. Không tự tạo layout label + radio.

**Sai:**

```tsx
// ❌ Tự tạo layout label + radio
<RadioGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
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
<RadioGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <FormControlLabel label="Lựa chọn A" control={<Radio value="a" />} />
  <FormControlLabel label="Lựa chọn B" control={<Radio value="b" />} />
</RadioGroup>
```

---

### RULE-RADIO-06: Sử dụng `row` trên RadioGroup cho layout ngang — không override flex direction

Mặc định `row=false` (vertical, flex-col). Truyền `row={true}` để chuyển sang layout ngang. Không override flex direction qua className.

**Sai:**

```tsx
// ❌ Override flex direction qua className
<RadioGroup
  className={cx("!flex-row !items-center")}
  value={selected}
  onChange={(e, newValue) => setSelected(newValue)}
>
  <Radio value="a" />
  <Radio value="b" />
</RadioGroup>
```

**Đúng:**

```tsx
// ✅
<RadioGroup row value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Radio value="a" />
  <Radio value="b" />
</RadioGroup>
```

---

### RULE-RADIO-07: `disabled` trên RadioGroup override disabled của từng children

Khi truyền `disabled` trên `RadioGroup`, tất cả children bị disabled bất kể disabled riêng của chúng. Chỉ truyền `disabled` trên từng child khi cần disable một số cụ thể.

**Sai:**

```tsx
// ❌ Truyền disabled trên từng child khi muốn disable tất cả
<RadioGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Radio value="a" disabled />
  <Radio value="b" disabled />
  <Radio value="c" disabled />
</RadioGroup>
```

**Đúng:**

```tsx
// ✅ Disable toàn bộ group
<RadioGroup disabled value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Radio value="a" />
  <Radio value="b" />
  <Radio value="c" />
</RadioGroup>

// ✅ Disable một số cụ thể
<RadioGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Radio value="a" />
  <Radio value="b" disabled />
  <Radio value="c" />
</RadioGroup>
```

---

### RULE-RADIO-08: Mỗi Radio trong RadioGroup phải có `value` unique

`RadioGroup` sử dụng `value` prop của mỗi child Radio để xác định checked state. Nếu thiếu `value` hoặc trùng `value`, chuyển đổi sẽ sai.

**Sai:**

```tsx
// ❌ Thiếu value — RadioGroup không thể xác định checked state
<RadioGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Radio />
  <Radio />
</RadioGroup>

// ❌ Value trùng — chọn một cái sẽ ảnh hưởng cả hai
<RadioGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Radio value="same" />
  <Radio value="same" />
</RadioGroup>
```

**Đúng:**

```tsx
// ✅ Mỗi radio có value unique
<RadioGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Radio value="option1" />
  <Radio value="option2" />
  <Radio value="option3" />
</RadioGroup>
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

### RULE-RADIO-10: `value` của Radio hỗ trợ `number | string | boolean` — không cần ép kiểu thủ công

Kiểu `RadioValue` đã được định nghĩa là `number | string | boolean`. Không cần ép kiểu giá trị trước khi truyền vào prop `value`.

**Sai:**

```tsx
// ❌ Ép kiểu thủ công không cần thiết
<Radio value={String(1)} checked={selected === "1"} onChange={handleChange} />
<Radio value={Number("2")} checked={selected === 2} onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅ Truyền trực tiếp giá trị — component tự xử lý
<Radio value={1} checked={selected === 1} onChange={handleChange} />
<Radio value="option1" checked={selected === "option1"} onChange={handleChange} />
<Radio value={true} checked={selected === true} onChange={handleChange} />
```
