---
title: Cách sử dụng component Cascader trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến mất dữ liệu chọn, dropdown không hoạt động, lỗi TypeScript generic, và hành vi chọn không nhất quán
tags: cascader, select, hierarchical, tree, dropdown, ui-kit
---

## Cách sử dụng component Cascader trong ui-kit

Đường dẫn component: `ui-kit/src/components/Cascader`

Props bắt buộc: `value`, `onChange`, `options`, `getValue`, `getLabel`.
Props tùy chọn: `singleSelect`, `searchable`, `filterBySearch`, `clearable`, `fullWidth`, `disabled`, `readOnly`, `error`, `helperText`, `color`, `noOptionText`, `tagVariant`, `tabIndex`, `onInputChange`, `onBlur`, `renderTag`, `renderLimitTag`, `renderOption`, `renderTagList`, `getOptionWidth`, `className`.

Kiểu dữ liệu option: `CascaderOption<T> = T & { children?: CascaderOption<T>[] }` — option dạng cây đệ quy.

Giá trị mặc định: `singleSelect=false`, `searchable=true`, `filterBySearch=true`, `noOptionText="No options"`.

---

### RULE-CASCADER-01: Luôn truyền đủ 5 props bắt buộc — `value`, `onChange`, `options`, `getValue`, `getLabel`

`Cascader` là controlled component. Phải truyền đủ cả 5 props: `value` (mảng các option đang chọn), `onChange` (callback nhận mảng mới), `options` (dữ liệu dạng cây), `getValue` (trích xuất value string), `getLabel` (trích xuất label hiển thị).

**Sai:**

```tsx
// ❌ Thiếu getValue và getLabel — TypeScript lỗi, component không biết cách đọc option
<Cascader
  value={selected}
  onChange={setSelected}
  options={departments}
/>
```

**Đúng:**

```tsx
// ✅
<Cascader
  value={selected}
  onChange={setSelected}
  options={departments}
  getValue={(opt) => opt.id}
  getLabel={(opt) => opt.name}
/>
```

---

### RULE-CASCADER-02: Sử dụng `singleSelect` để chuyển sang chế độ chọn một — không tự quản lý logic single select

Mặc định `singleSelect=false` (multi-select dùng Checkbox). Khi `singleSelect=true`, component tự chuyển sang Radio và gán `value` là mảng một phần tử. Không tự viết logic giới hạn số lượng chọn.

**Sai:**

```tsx
// ❌ Tự giới hạn mảng value — logic thừa và dễ lỗi
<Cascader
  value={selected}
  onChange={(newValue) => setSelected(newValue.slice(-1))}
  options={departments}
  getValue={(opt) => opt.id}
  getLabel={(opt) => opt.name}
/>
```

**Đúng:**

```tsx
// ✅ Component tự xử lý single select bên trong
<Cascader
  singleSelect
  value={selected}
  onChange={setSelected}
  options={departments}
  getValue={(opt) => opt.id}
  getLabel={(opt) => opt.name}
/>
```

---

### RULE-CASCADER-03: Option có `children` sẽ mở rộng cột con — không chọn trực tiếp

Khi click option có `children`, component mở rộng cột cascade tiếp theo thay vì toggle chọn. Chỉ option lá (không có `children`) mới được chọn/bỏ chọn khi click. Không đặt logic chọn lên option cha có children.

**Sai:**

```tsx
// ❌ Đặt option cha không có children vào cấp đầu khi muốn nó chọn được
// nhưng lại gán children rỗng — children rỗng (undefined) mới đúng
const options = [
  {
    id: "1",
    name: "Phòng A",
    children: [], // ❌ Mảng rỗng vẫn truthy — option sẽ không chọn được
  },
];
```

**Đúng:**

```tsx
// ✅ Option lá không có children — click sẽ chọn/bỏ chọn
const options = [
  {
    id: "1",
    name: "Phòng A",
    children: [
      { id: "1-1", name: "Đội 1" }, // Lá — chọn được
      { id: "1-2", name: "Đội 2" }, // Lá — chọn được
    ],
  },
  { id: "2", name: "Phòng B" }, // Lá — chọn được
];
```

---

### RULE-CASCADER-04: Sử dụng `fullWidth` thay vì override className cho chiều rộng

Chiều rộng mặc định là `190px`. Sử dụng `fullWidth` để component chiếm toàn bộ chiều rộng parent. Không tự override width qua `className`.

**Sai:**

```tsx
// ❌ Override width qua className — dễ conflict với logic nội bộ
<Cascader
  className={cx("w-full")}
  value={selected}
  onChange={setSelected}
  options={options}
  getValue={(opt) => opt.id}
  getLabel={(opt) => opt.name}
/>
```

**Đúng:**

```tsx
// ✅
<Cascader
  fullWidth
  value={selected}
  onChange={setSelected}
  options={options}
  getValue={(opt) => opt.id}
  getLabel={(opt) => opt.name}
/>
```

---

### RULE-CASCADER-05: Sử dụng `error` kết hợp `helperText` cho validation — không tự thêm HelperText bên ngoài

Component đã tích hợp sẵn `HelperText` bên dưới. Khi `error=true`, border chuyển đỏ và `helperText` tự hiển thị với color error. Không tự render `HelperText` bên ngoài component.

**Sai:**

```tsx
// ❌ Tự thêm HelperText bên ngoài — bị duplicate
<div>
  <Cascader
    value={selected}
    onChange={setSelected}
    options={options}
    getValue={(opt) => opt.id}
    getLabel={(opt) => opt.name}
  />
  <HelperText color="error">Vui lòng chọn ít nhất 1 mục</HelperText>
</div>
```

**Đúng:**

```tsx
// ✅
<Cascader
  error={hasError}
  helperText="Vui lòng chọn ít nhất 1 mục"
  value={selected}
  onChange={setSelected}
  options={options}
  getValue={(opt) => opt.id}
  getLabel={(opt) => opt.name}
/>
```

---

### RULE-CASCADER-06: Sử dụng `clearable` để hiển thị nút xoá — không tự build clear button

Khi `clearable=true`, nút xoá (×) tự hiển thị khi hover hoặc focus và component có giá trị. Click nút xoá gọi `onChange([])`. Không tự build nút clear bên ngoài.

**Sai:**

```tsx
// ❌ Tự build nút clear bên ngoài
<div className={cx("flex items-center gap-[0.5rem]")}>
  <Cascader
    value={selected}
    onChange={setSelected}
    options={options}
    getValue={(opt) => opt.id}
    getLabel={(opt) => opt.name}
  />
  <button onClick={() => setSelected([])}>Xoá</button>
</div>
```

**Đúng:**

```tsx
// ✅
<Cascader
  clearable
  value={selected}
  onChange={setSelected}
  options={options}
  getValue={(opt) => opt.id}
  getLabel={(opt) => opt.name}
/>
```

---

### RULE-CASCADER-07: Sử dụng `searchable={false}` để tắt tìm kiếm — không ẩn bằng CSS

Mặc định `searchable=true`. Khi không cần tìm kiếm, truyền `searchable={false}`. Prop `filterBySearch` kiểm soát việc lọc option theo search (mặc định `true`). Nếu muốn search input nhưng tự lọc bên ngoài, dùng `filterBySearch={false}` kết hợp `onInputChange`.

**Sai:**

```tsx
// ❌ Ẩn search bằng CSS — search input vẫn tồn tại, autoFocus vẫn kích hoạt
<Cascader
  className={cx("[&_.stringee-cascader-search]:hidden")}
  value={selected}
  onChange={setSelected}
  options={options}
  getValue={(opt) => opt.id}
  getLabel={(opt) => opt.name}
/>
```

**Đúng:**

```tsx
// ✅ Tắt search hoàn toàn
<Cascader
  searchable={false}
  value={selected}
  onChange={setSelected}
  options={options}
  getValue={(opt) => opt.id}
  getLabel={(opt) => opt.name}
/>

// ✅ Giữ search input nhưng tự lọc bên ngoài
<Cascader
  filterBySearch={false}
  onInputChange={(e) => handleCustomFilter(e.target.value)}
  value={selected}
  onChange={setSelected}
  options={filteredOptions}
  getValue={(opt) => opt.id}
  getLabel={(opt) => opt.name}
/>
```

---

### RULE-CASCADER-08: Sử dụng `renderOption` để tuỳ chỉnh hiển thị option — không override CSS nội bộ

Để thay đổi cách hiển thị mỗi option trong dropdown, sử dụng prop `renderOption`. Không override CSS selector nội bộ để thay đổi giao diện option.

**Sai:**

```tsx
// ❌ Override CSS nội bộ — dễ vỡ khi component cập nhật
<Cascader
  className={cx("[&_.stringee-cascader-option]:text-red-500")}
  value={selected}
  onChange={setSelected}
  options={options}
  getValue={(opt) => opt.id}
  getLabel={(opt) => opt.name}
/>
```

**Đúng:**

```tsx
// ✅
<Cascader
  value={selected}
  onChange={setSelected}
  options={options}
  getValue={(opt) => opt.id}
  getLabel={(opt) => opt.name}
  renderOption={(opt) => (
    <div className={cx("flex items-center gap-[0.5rem]")}>
      <FontAwesomeIcon icon={faFolder} />
      <span>{opt.name}</span>
    </div>
  )}
/>
```

---

### RULE-CASCADER-09: `disabled` và `readOnly` có hành vi khác nhau — không nhầm lẫn

`disabled` làm component xám, không tương tác được, border `primary-light-94`. `readOnly` làm component trong suốt, không border, vẫn hiển thị giá trị nhưng không mở dropdown. Cả hai đều vô hiệu hoá Popper.

**Sai:**

```tsx
// ❌ Dùng disabled khi chỉ muốn hiển thị giá trị
<Cascader
  disabled
  value={selected}
  onChange={setSelected}
  options={options}
  getValue={(opt) => opt.id}
  getLabel={(opt) => opt.name}
/>
```

**Đúng:**

```tsx
// ✅ Dùng readOnly để hiển thị giá trị mà không cho chỉnh sửa
<Cascader
  readOnly
  value={selected}
  onChange={setSelected}
  options={options}
  getValue={(opt) => opt.id}
  getLabel={(opt) => opt.name}
/>

// ✅ Dùng disabled khi field thực sự bị vô hiệu hoá
<Cascader
  disabled
  value={selected}
  onChange={setSelected}
  options={options}
  getValue={(opt) => opt.id}
  getLabel={(opt) => opt.name}
/>
```

---

### RULE-CASCADER-10: Sử dụng `renderTag` và `renderLimitTag` để tuỳ chỉnh tag — không tự build tag list bên ngoài

Component tự render tag cho mỗi option đã chọn và tự xử lý overflow (+n). Sử dụng `renderTag` để tuỳ chỉnh từng tag, `renderLimitTag` để tuỳ chỉnh tag overflow. Chỉ dùng `renderTagList` khi cần thay thế toàn bộ tag list.

**Sai:**

```tsx
// ❌ Tự build tag list bên ngoài — mất logic overflow tự động
<div className={cx("flex gap-[0.25rem]")}>
  {selected.map((opt) => (
    <Tag key={opt.id}>{opt.name}</Tag>
  ))}
</div>
<Cascader
  value={selected}
  onChange={setSelected}
  options={options}
  getValue={(opt) => opt.id}
  getLabel={(opt) => opt.name}
/>
```

**Đúng:**

```tsx
// ✅ Tuỳ chỉnh từng tag
<Cascader
  value={selected}
  onChange={setSelected}
  options={options}
  getValue={(opt) => opt.id}
  getLabel={(opt) => opt.name}
  renderTag={(opt, index, isFocused, onDelete) => (
    <Tag
      key={opt.id}
      variant="outlined"
      showDeleteIcon={isFocused}
      onDelete={onDelete}
    >
      {opt.name}
    </Tag>
  )}
  renderLimitTag={(count) => <Tag variant="outlined">+{count}</Tag>}
/>
```
