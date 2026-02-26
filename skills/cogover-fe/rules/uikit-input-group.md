---
title: Cách sử dụng InputGroup Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng InputGroup sai dẫn đến việc layout không đúng, Select bên trái không hiển thị đúng style, và children bên phải bị vỡ giao diện
tags: input-group, select, combine, group, ui-kit
---

## Cách sử dụng InputGroup Component của ui-kit

Đường dẫn component: `ui-kit/src/components/InputGroup`

Props: `groupSelectProps`, `groupSelectWidth`, `fullWidth`, `children`.

InputGroup là component kết hợp một Select ở bên trái với một input/component bất kỳ ở bên phải. Select bên trái có style riêng (bo góc phải bị loại bỏ, nền xám) và children bên phải sẽ bị loại bỏ bo góc trái để tạo thành một khối liền mạch.

---

### RULE-INPUTGROUP-01: Truyền `groupSelectProps` đúng cách — phải bao gồm `value`, `onChange`, `options`

`groupSelectProps` nhận tất cả props của `Select` component. Bắt buộc phải truyền ít nhất `value`, `onChange` và `options` (hoặc `groupOptions` nếu dùng mode group).

**Sai:**

```tsx
// ❌ Thiếu onChange và options — Select sẽ không hoạt động
<InputGroup groupSelectProps={{ value: null }}>
  <TextField />
</InputGroup>
```

**Đúng:**

```tsx
// ✅ Truyền đầy đủ các props bắt buộc cho Select
<InputGroup
  groupSelectProps={{
    value: selectedOption,
    onChange: setSelectedOption,
    options: options,
    getLabel: (option) => option.label,
    getValue: (option) => option.value,
  }}
>
  <TextField />
</InputGroup>
```

---

### RULE-INPUTGROUP-02: Sử dụng `groupSelectWidth` để điều chỉnh chiều rộng Select, không dùng CSS thủ công

Chiều rộng mặc định của Select bên trái là `108px`. Khi cần thay đổi, sử dụng prop `groupSelectWidth` thay vì override CSS.

**Sai:**

```tsx
// ❌ Override CSS thủ công — không đồng bộ với logic component
<InputGroup
  groupSelectProps={{ value, onChange, options }}
>
  <TextField />
</InputGroup>
// Rồi dùng CSS: .stringee-input-group-item { width: 85px !important; }
```

**Đúng:**

```tsx
// ✅ Sử dụng prop groupSelectWidth
<InputGroup
  groupSelectWidth={85}
  groupSelectProps={{
    value: selectedType,
    onChange: setSelectedType,
    options: groupOptions,
  }}
>
  <TextField />
</InputGroup>
```

---

### RULE-INPUTGROUP-03: Không truyền rõ ràng giá trị mặc định cho `groupSelectWidth` và `fullWidth`

`groupSelectWidth` mặc định là `108` và `fullWidth` mặc định là `false`. Chỉ truyền khi cần thay đổi giá trị mặc định.

**Sai:**

```tsx
// ❌ Thừa — đã là giá trị mặc định
<InputGroup
  groupSelectWidth={108}
  fullWidth={false}
  groupSelectProps={{ value, onChange, options }}
>
  <TextField />
</InputGroup>
```

**Đúng:**

```tsx
// ✅ Bỏ qua khi dùng giá trị mặc định
<InputGroup groupSelectProps={{ value, onChange, options }}>
  <TextField />
</InputGroup>

// ✅ Chỉ truyền khi cần thay đổi
<InputGroup fullWidth groupSelectWidth={85} groupSelectProps={{ value, onChange, options }}>
  <TextField />
</InputGroup>
```

---

### RULE-INPUTGROUP-04: Sử dụng `fullWidth` khi cần InputGroup chiếm toàn bộ chiều rộng

Khi `fullWidth={true}`, container sẽ có `w-full` và phần children bên phải sẽ có `flex-1 w-0` để tự động chiếm phần còn lại.

**Sai:**

```tsx
// ❌ Thêm className w-full thủ công — không kích hoạt flex-1 cho children
<div className={cx("w-full")}>
  <InputGroup groupSelectProps={{ value, onChange, options }}>
    <TextField />
  </InputGroup>
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng prop fullWidth
<InputGroup
  fullWidth
  groupSelectProps={{
    value: selectedType,
    onChange: setSelectedType,
    options: groupOptions,
  }}
>
  <TextField />
</InputGroup>
```

---

### RULE-INPUTGROUP-05: Children phải là component input có hỗ trợ class `stringee-input-root` hoặc `stringee-tags-list`

InputGroup áp dụng style tự động (loại bỏ bo góc trái) thông qua CSS selector `.stringee-input-item .stringee-input-root` và `.stringee-input-item .stringee-tags-list`. Children nên là các component từ ui-kit như `TextField`, `Select`, `MultipleSelect` hoặc các component có sử dụng class tương ứng.

**Sai:**

```tsx
// ❌ Input HTML thuần — không có class stringee-input-root, style sẽ không được áp dụng
<InputGroup groupSelectProps={{ value, onChange, options }}>
  <input type="text" />
</InputGroup>
```

**Đúng:**

```tsx
// ✅ Sử dụng TextField từ ui-kit
<InputGroup groupSelectProps={{ value, onChange, options }}>
  <TextField value={text} onChange={handleChange} />
</InputGroup>

// ✅ Sử dụng MultipleSelect từ ui-kit
<InputGroup groupSelectProps={{ value, onChange, options }}>
  <MultipleSelect
    value={selectedValues}
    onChange={setSelectedValues}
    options={multiOptions}
    getLabel={(option) => option.label}
    getValue={(option) => option.value}
  />
</InputGroup>
```

---

### RULE-INPUTGROUP-06: Không truyền `minWidthPopper` cho `groupSelectProps` trừ khi cần ghi đè

InputGroup đã tự động truyền `minWidthPopper='200'` và `fullWidth` cho Select bên trong. Không cần truyền lại trừ khi cần ghi đè giá trị khác.

**Sai:**

```tsx
// ❌ Thừa — đã được truyền mặc định bên trong component
<InputGroup
  groupSelectProps={{
    value,
    onChange,
    options,
    minWidthPopper: "200",
    fullWidth: true,
  }}
>
  <TextField />
</InputGroup>
```

**Đúng:**

```tsx
// ✅ Bỏ qua khi dùng giá trị mặc định
<InputGroup groupSelectProps={{ value, onChange, options }}>
  <TextField />
</InputGroup>

// ✅ Chỉ truyền khi cần ghi đè giá trị khác
<InputGroup
  groupSelectProps={{
    value,
    onChange,
    options,
    minWidthPopper: "100",
  }}
>
  <TextField />
</InputGroup>
```

---

## Tham chiếu props

| Prop | Kiểu | Mặc định | Mô tả |
|------|------|----------|-------|
| `groupSelectProps` | `SelectProps<T>` | _(bắt buộc)_ | Props truyền cho Select bên trái |
| `groupSelectWidth` | `number` | `108` | Chiều rộng (px) của Select bên trái |
| `fullWidth` | `boolean` | `false` | Chiếm toàn bộ chiều rộng container |
| `children` | `React.ReactNode` | — | Component input hiển thị bên phải |

## Tham chiếu style

| Phần tử | Style áp dụng |
|---------|---------------|
| Select bên trái | `rounded-r-none`, `bg-gray-5`, `border-gray-5` |
| Children bên phải | `rounded-l-none` |
| Container (fullWidth) | `w-full` |
| Children (fullWidth) | `flex-1 w-0` |
