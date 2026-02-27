---
title: Cách sử dụng component Checkbox và CheckboxGroup trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến trạng thái checked không đồng bộ, indeterminate hiển thị sai, CheckboxGroup không toggle đúng giá trị, và lỗi accessibility
tags: checkbox, checkbox-group, form, input, ui-kit, form-control-label
---

## Cách sử dụng component Checkbox và CheckboxGroup trong ui-kit

Đường dẫn component:
- `ui-kit/src/components/Checkbox`
- `ui-kit/src/components/CheckboxGroup`
- `ui-kit/src/components/FormControlLabel` (dùng kèm CheckboxGroup)

**Checkbox** props: `checked`, `onChange`, `disabled`, `indeterminate`, `size`, `value`, `className`. Extends `Omit<InputHTMLAttributes, "size" | "defaultChecked" | "value">`.
**CheckboxGroup** props: `value` (T[]), `onChange` (e, value[]) => void, `row`, `disabled`, `className`.
**FormControlLabel** props: `label`, `control`, `labelPlacement`, `labelAlign`, `disabled`, `disabledLabel`, `usingHtmlFor`, `className`, `classNameLabel`.

Giá trị mặc định: Checkbox `size="medium"`, `indeterminate=false`. CheckboxGroup `row=false`. FormControlLabel `labelPlacement="right"`, `disabledLabel=true`, `usingHtmlFor=true`.

---

### RULE-CHECKBOX-01: Pair `checked` với `onChange` trong controlled mode — omit `checked` cho uncontrolled

Khi truyền `checked`, component chạy controlled mode — state do bạn quản lý. Khi không truyền `checked`, component tự quản lý state nội bộ. Không truyền `checked` mà thiếu `onChange` (sẽ không cập nhật được).

**Sai:**

```tsx
// ❌ Truyền checked nhưng thiếu onChange — checkbox bị "đóng băng"
<Checkbox checked={isAgree} />

// ❌ Dùng defaultChecked — prop này đã bị omit khỏi type
<Checkbox defaultChecked={true} onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅ Controlled mode
<Checkbox checked={isAgree} onChange={(e) => setIsAgree(e.target.checked)} />

// ✅ Uncontrolled mode — component tự quản lý state
<Checkbox onChange={(e) => handleChange(e)} />
```

---

### RULE-CHECKBOX-02: Sử dụng `indeterminate` cho trạng thái "chọn một phần" — không tự build icon

`indeterminate` hiển thị icon dấu trừ (−) khi checkbox chưa checked. Khi `checked=true`, icon checked luôn hiển thị bất kể `indeterminate`. Không tự tạo icon indeterminate bằng cách khác.

**Sai:**

```tsx
// ❌ Tự build icon indeterminate
<Checkbox
  checked={someChecked}
  onChange={handleChange}
  className={cx({ "[&_svg]:hidden": isPartial })}
/>
{isPartial && <MinusIcon />}
```

**Đúng:**

```tsx
// ✅ Indeterminate khi một số con được chọn nhưng chưa hết
<Checkbox
  checked={allChecked}
  indeterminate={!allChecked && someChecked}
  onChange={handleToggleAll}
/>
```

---

### RULE-CHECKBOX-03: Sử dụng `size` cho kích thước — không override width/height qua className

Ba kích thước có sẵn: `"large"` (24px), `"medium"` (20px, mặc định), `"small"` (16px). Không tự override kích thước qua className.

**Sai:**

```tsx
// ❌ Override kích thước qua className — phá vỡ border-radius và icon scaling
<Checkbox checked={value} onChange={handleChange} className={cx("w-[2rem] h-[2rem]")} />
```

**Đúng:**

```tsx
// ✅
<Checkbox checked={value} onChange={handleChange} size="large" />
<Checkbox checked={value} onChange={handleChange} size="small" />
```

---

### RULE-CHECKBOX-04: Không truyền `type` — component luôn là checkbox

Input type luôn được set cứng là `"checkbox"` bên trong component. Không truyền `type` prop.

**Sai:**

```tsx
// ❌ Thừa — type luôn là "checkbox"
<Checkbox type="checkbox" checked={value} onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅
<Checkbox checked={value} onChange={handleChange} />
```

---

### RULE-CHECKBOX-05: Sử dụng `CheckboxGroup` để quản lý nhóm checkbox — không tự viết logic toggle mảng

`CheckboxGroup` nhận `value` (mảng T[]) và `onChange(e, newValue[])`. Component tự xử lý toggle item trong mảng. Không tự viết logic thêm/xoá item.

**Sai:**

```tsx
// ❌ Tự quản lý logic toggle mảng cho từng checkbox
<div className={cx("flex flex-col gap-[0.75rem]")}>
  <Checkbox
    value="a"
    checked={selected.includes("a")}
    onChange={(e) => {
      if (e.target.checked) setSelected([...selected, "a"]);
      else setSelected(selected.filter((v) => v !== "a"));
    }}
  />
  <Checkbox
    value="b"
    checked={selected.includes("b")}
    onChange={(e) => {
      if (e.target.checked) setSelected([...selected, "b"]);
      else setSelected(selected.filter((v) => v !== "b"));
    }}
  />
</div>
```

**Đúng:**

```tsx
// ✅ CheckboxGroup tự xử lý toggle
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" />
  <Checkbox value="b" />
</CheckboxGroup>
```

---

### RULE-CHECKBOX-06: Sử dụng `FormControlLabel` để thêm label cho checkbox trong CheckboxGroup

Khi cần label bên cạnh checkbox trong `CheckboxGroup`, wrap bằng `FormControlLabel` với prop `control`. Không tự tạo layout label + checkbox.

**Sai:**

```tsx
// ❌ Tự tạo layout label + checkbox
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <div className={cx("flex items-center gap-[0.5rem]")}>
    <Checkbox value="a" />
    <span>Lựa chọn A</span>
  </div>
  <div className={cx("flex items-center gap-[0.5rem]")}>
    <Checkbox value="b" />
    <span>Lựa chọn B</span>
  </div>
</CheckboxGroup>
```

**Đúng:**

```tsx
// ✅ FormControlLabel tự xử lý layout, htmlFor, và disabled state
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <FormControlLabel label="Lựa chọn A" control={<Checkbox value="a" />} />
  <FormControlLabel label="Lựa chọn B" control={<Checkbox value="b" />} />
</CheckboxGroup>
```

---

### RULE-CHECKBOX-07: Sử dụng `row` trên CheckboxGroup cho layout ngang — không override flex direction

Mặc định `row=false` (vertical, flex-col). Truyền `row={true}` để chuyển sang layout ngang. Không override flex direction qua className.

**Sai:**

```tsx
// ❌ Override flex direction qua className
<CheckboxGroup
  className={cx("!flex-row !items-center")}
  value={selected}
  onChange={(e, newValue) => setSelected(newValue)}
>
  <Checkbox value="a" />
  <Checkbox value="b" />
</CheckboxGroup>
```

**Đúng:**

```tsx
// ✅
<CheckboxGroup row value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" />
  <Checkbox value="b" />
</CheckboxGroup>
```

---

### RULE-CHECKBOX-08: `disabled` trên CheckboxGroup override disabled của từng children

Khi truyền `disabled` trên `CheckboxGroup`, tất cả children bị disabled bất kể disabled riêng của chúng. Chỉ truyền `disabled` trên từng child khi cần disable một số cụ thể.

**Sai:**

```tsx
// ❌ Truyền disabled trên từng child khi muốn disable tất cả
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" disabled />
  <Checkbox value="b" disabled />
  <Checkbox value="c" disabled />
</CheckboxGroup>
```

**Đúng:**

```tsx
// ✅ Disable toàn bộ group
<CheckboxGroup disabled value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" />
  <Checkbox value="b" />
  <Checkbox value="c" />
</CheckboxGroup>

// ✅ Disable một số cụ thể
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" />
  <Checkbox value="b" disabled />
  <Checkbox value="c" />
</CheckboxGroup>
```

---

### RULE-CHECKBOX-09: Mỗi Checkbox trong CheckboxGroup phải có `value` unique

`CheckboxGroup` sử dụng `value` prop của mỗi child Checkbox làm key và để xác định checked state. Nếu thiếu `value` hoặc trùng `value`, toggle sẽ sai.

**Sai:**

```tsx
// ❌ Thiếu value — CheckboxGroup không thể xác định checked state
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox />
  <Checkbox />
</CheckboxGroup>

// ❌ Value trùng — toggle một cái sẽ ảnh hưởng cả hai
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="same" />
  <Checkbox value="same" />
</CheckboxGroup>
```

**Đúng:**

```tsx
// ✅ Mỗi checkbox có value unique
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="option1" />
  <Checkbox value="option2" />
  <Checkbox value="option3" />
</CheckboxGroup>
```

---

### RULE-CHECKBOX-10: Sử dụng `labelPlacement` trên FormControlLabel để đổi vị trí label — không đảo thứ tự JSX

`FormControlLabel` hỗ trợ `labelPlacement`: `"right"` (mặc định), `"left"`, `"top"`, `"bottom"`. Không đảo thứ tự control và label trong JSX.

**Sai:**

```tsx
// ❌ Đảo thứ tự JSX để label ở bên trái
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
```
