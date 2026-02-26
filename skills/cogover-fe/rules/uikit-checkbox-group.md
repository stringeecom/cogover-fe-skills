---
title: Cách sử dụng component CheckboxGroup trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến toggle mảng giá trị không đúng, layout bị lỗi, disabled state không đồng bộ, và mất khả năng quản lý nhóm checkbox
tags: checkbox-group, checkbox, form, ui-kit, form-control-label
---

## Cách sử dụng component CheckboxGroup trong ui-kit

Đường dẫn component: `ui-kit/src/components/CheckboxGroup`

**CheckboxGroup** props: `value` (T[]), `onChange` ((e, value: T[]) => void), `row` (boolean), `disabled` (boolean), `className` (string), `name` (string). Hỗ trợ spread thêm các HTML attributes khác qua `[key: string]: unknown`.

Giá trị mặc định: `row=false` (layout dọc).

Children có thể là `Checkbox` trực tiếp hoặc `FormControlLabel` với `control={<Checkbox />}`. Component tự xử lý logic toggle thêm/xoá item trong mảng `value`.

---

### RULE-CBXGRP-01: Luôn truyền `value` (mảng) và `onChange` — CheckboxGroup là controlled component

`CheckboxGroup` yêu cầu `value` là mảng T[] và `onChange` nhận `(e, newValue: T[])`. Không có uncontrolled mode. Không tự quản lý checked state cho từng Checkbox bên trong.

**Sai:**

```tsx
// ❌ Thiếu value — component sẽ lỗi vì gọi value.includes()
<CheckboxGroup onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" />
  <Checkbox value="b" />
</CheckboxGroup>

// ❌ Tự truyền checked cho từng Checkbox — CheckboxGroup đã tự xử lý checked
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" checked={selected.includes("a")} />
  <Checkbox value="b" checked={selected.includes("b")} />
</CheckboxGroup>
```

**Đúng:**

```tsx
// ✅ Chỉ cần truyền value và onChange cho CheckboxGroup
const [selected, setSelected] = useState<string[]>([]);

<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" />
  <Checkbox value="b" />
</CheckboxGroup>
```

---

### RULE-CBXGRP-02: Không tự viết logic toggle mảng — CheckboxGroup đã xử lý sẵn

Component tự xử lý thêm item khi checked và xoá item khi unchecked. Callback `onChange` trả về mảng mới đã được toggle. Không tự viết logic `filter` hoặc spread mảng.

**Sai:**

```tsx
// ❌ Tự viết logic toggle cho từng checkbox riêng lẻ
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
// ✅ CheckboxGroup tự toggle mảng
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" />
  <Checkbox value="b" />
</CheckboxGroup>
```

---

### RULE-CBXGRP-03: Mỗi Checkbox con phải có `value` unique — dùng làm key và xác định checked state

CheckboxGroup dùng `value` prop của mỗi child Checkbox (hoặc `control` Checkbox trong FormControlLabel) làm key trong React và để so sánh với mảng `value` của group. Thiếu `value` hoặc trùng `value` sẽ gây toggle sai.

**Sai:**

```tsx
// ❌ Thiếu value — không xác định được checked state
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox />
  <Checkbox />
</CheckboxGroup>

// ❌ Value trùng — toggle một cái ảnh hưởng cả hai
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

### RULE-CBXGRP-04: Sử dụng `row` để chuyển layout ngang — không override flex direction qua className

Mặc định `row=false` hiển thị dọc (`flex-col`). Truyền `row={true}` để chuyển sang layout ngang (`items-center`). Không dùng className để override flex direction.

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
// ✅ Dùng prop row
<CheckboxGroup row value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" />
  <Checkbox value="b" />
</CheckboxGroup>
```

---

### RULE-CBXGRP-05: `disabled` trên CheckboxGroup override disabled của từng children

Khi truyền `disabled` trên CheckboxGroup, tất cả children đều bị disabled bất kể disabled riêng của chúng. Khi không truyền `disabled` (undefined), disabled state được xác định bởi từng child. Chỉ truyền `disabled` trên group khi muốn disable toàn bộ.

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

// ✅ Disable một số cụ thể — không truyền disabled trên group
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" />
  <Checkbox value="b" disabled />
  <Checkbox value="c" />
</CheckboxGroup>
```

---

### RULE-CBXGRP-06: Sử dụng `FormControlLabel` để thêm label — không tự tạo layout label + checkbox

Khi cần label bên cạnh checkbox trong CheckboxGroup, dùng `FormControlLabel` với prop `control`. CheckboxGroup hỗ trợ cả hai loại children: Checkbox trực tiếp và FormControlLabel. Khi dùng FormControlLabel, `value` phải nằm trên Checkbox bên trong `control`.

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

### RULE-CBXGRP-07: Không tự wrap container div — CheckboxGroup đã render div với flex layout

CheckboxGroup render một `<div>` với `flex gap-[0.75rem]` và hỗ trợ spread thêm HTML attributes. Không wrap thêm div bên ngoài cho mục đích layout.

**Sai:**

```tsx
// ❌ Wrap thừa div — CheckboxGroup đã là flex container
<div className={cx("flex flex-col gap-[0.75rem]")}>
  <CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
    <Checkbox value="a" />
    <Checkbox value="b" />
  </CheckboxGroup>
</div>
```

**Đúng:**

```tsx
// ✅ Truyền className trực tiếp nếu cần thêm style
<CheckboxGroup
  value={selected}
  onChange={(e, newValue) => setSelected(newValue)}
  className={cx("mt-[1rem]")}
>
  <Checkbox value="a" />
  <Checkbox value="b" />
</CheckboxGroup>
```

---

### RULE-CBXGRP-08: Generic type T được suy luận từ `value` — đảm bảo kiểu nhất quán

CheckboxGroup là generic component `CheckboxGroup<T>`. Kiểu T được suy luận từ mảng `value`. Đảm bảo value của các Checkbox con cùng kiểu với phần tử trong mảng `value`.

**Sai:**

```tsx
// ❌ Kiểu không nhất quán — value là string[] nhưng Checkbox value là number
const [selected, setSelected] = useState<string[]>([]);

<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value={1} />
  <Checkbox value={2} />
</CheckboxGroup>
```

**Đúng:**

```tsx
// ✅ Kiểu nhất quán — value là number[], Checkbox value cũng là number
const [selected, setSelected] = useState<number[]>([]);

<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value={1} />
  <Checkbox value={2} />
</CheckboxGroup>

// ✅ Kiểu string
const [selected, setSelected] = useState<string[]>([]);

<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="option1" />
  <Checkbox value="option2" />
</CheckboxGroup>
```
