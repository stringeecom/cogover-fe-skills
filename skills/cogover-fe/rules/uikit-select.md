---
title: Cách sử dụng component Select trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến mất giá trị đã chọn, dropdown không hoạt động, lỗi TypeScript generic, và hành vi chọn không nhất quán
tags: select, dropdown, combobox, searchable, clearable, group, ui-kit
---

## Cách sử dụng component Select trong ui-kit

Đường dẫn component: `ui-kit/src/components/Select`

Props bắt buộc: `value`, `onChange`.
Props tuỳ chọn: `options`, `groupOptions`, `mode`, `searchable`, `clearable`, `filterBySearch`, `showCheckIcon`, `showTooltip`, `showValueTooltip`, `showBorderOnBlur`, `appendToBody`, `hidePopperOnClear`, `hidePopperOnSelect`, `disabledKeyboard`, `disabledValues`, `height`, `minWidthPopper`, `noOptionText`, `showMoreText`, `showArrowIcon`, `showOperatorSelect`, `operator`, `operatorOptions`, `onChangeOperator`, `onInputChange`, `onScrollToBottom`, `onBlur`, `inputRef`, `optionListClassName`, `popperProps`, `renderOption`, `renderValue`, `getLabel`, `getValue`, `isEqual`, `getKey`, `renderStartAction`, `renderEndAction`, `startAdornment`, `endAdornment`, `alwaysShowAdornment`, `helperText`, `color`, `error`, `className`.

Kế thừa từ `TextFieldProps` (ngoại trừ `value`, `onChange`, `onBlur`) nên hỗ trợ: `fullWidth`, `disabled`, `readOnly`, `placeholder`, `autoFocus`, `name`, `tabIndex`, `onFocus`, `onKeyDown`, và các thuộc tính HTML input khác.

Giá trị mặc định: `mode="default"`, `height=40`, `searchable=undefined`, `clearable=undefined`, `filterBySearch=true`, `showCheckIcon=true`, `showTooltip=true`, `showValueTooltip=true`, `showBorderOnBlur=true`, `appendToBody=true`, `hidePopperOnClear=false`, `hidePopperOnSelect=true`, `disabledKeyboard=false`, `showArrowIcon=true`, `alwaysShowAdornment=false`.

---

### RULE-SELECT-01: Select là controlled component — phải truyền `value` và `onChange`

`Select` là controlled component. `value` là option đang được chọn (hoặc `null` khi chưa chọn). `onChange` nhận về option mới hoặc `null` khi clear. Không tự quản lý state bên trong component.

**Sai:**

```tsx
// ❌ Thiếu value và onChange — component không hoạt động
<Select options={users} />
```

**Đúng:**

```tsx
// ✅ Controlled component với value và onChange
const [selected, setSelected] = useState<User | null>(null);

<Select
  value={selected}
  onChange={setSelected}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

---

### RULE-SELECT-02: Sử dụng `getLabel` và `getValue` khi option không có dạng `{ label, value }`

Mặc định `getLabel` đọc `option.label` và `getValue` đọc `option.value` (thông qua `object-path`). Khi kiểu dữ liệu option khác, phải truyền `getLabel` và `getValue` để component biết cách đọc dữ liệu.

**Sai:**

```tsx
// ❌ Option có dạng { id, name } nhưng không truyền getLabel/getValue
// Component sẽ không đọc được label và value
<Select
  value={selected}
  onChange={setSelected}
  options={users}
/>
```

**Đúng:**

```tsx
// ✅ Truyền getLabel và getValue phù hợp với kiểu dữ liệu
<Select
  value={selected}
  onChange={setSelected}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

---

### RULE-SELECT-03: Sử dụng `mode="group"` kết hợp `groupOptions` để hiển thị option theo nhóm — không tự build nhóm bên ngoài

Khi cần nhóm các option, sử dụng `mode="group"` và truyền `groupOptions` (mảng các nhóm với `key`, `label`, `children`). Component tự render phân chia nhóm với đường kẻ và tiêu đề. Không tự build giao diện nhóm bên ngoài.

**Sai:**

```tsx
// ❌ Tự build nhóm bên ngoài — mất logic focus, keyboard, scroll
<div>
  <h3>Nhóm A</h3>
  <Select options={groupA} value={selected} onChange={setSelected} />
  <h3>Nhóm B</h3>
  <Select options={groupB} value={selected} onChange={setSelected} />
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng mode="group" và groupOptions
<Select
  mode="group"
  groupOptions={[
    { key: "group-a", label: "Nhóm A", children: groupA },
    { key: "group-b", label: "Nhóm B", children: groupB },
  ]}
  value={selected}
  onChange={setSelected}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

---

### RULE-SELECT-04: Sử dụng `searchable` để bật tìm kiếm — không tự build search input bên ngoài

Khi `searchable=true`, input cho phép người dùng gõ để lọc option. Khi focus, giá trị hiện tại sẽ bị xoá để người dùng gõ từ khoá. Khi blur, giá trị hiện tại được phục hồi. Không tự build search input bên ngoài Select.

**Sai:**

```tsx
// ❌ Tự build search input bên ngoài — mất đồng bộ với Select
<TextField value={searchText} onChange={(e) => setSearchText(e.target.value)} />
<Select
  value={selected}
  onChange={setSelected}
  options={filteredOptions}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng searchable — component tự xử lý search
<Select
  searchable
  value={selected}
  onChange={setSelected}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

---

### RULE-SELECT-05: Sử dụng `filterBySearch={false}` khi muốn tự lọc option bên ngoài

Mặc định `filterBySearch=true` — component tự lọc option theo search text. Khi cần tự lọc (ví dụ: gọi API tìm kiếm), dùng `filterBySearch={false}` kết hợp `onInputChange` để lấy search text và tự cập nhật `options`.

**Sai:**

```tsx
// ❌ Bật searchable nhưng tự lọc options bằng state — bị lọc 2 lần
<Select
  searchable
  value={selected}
  onChange={setSelected}
  options={options.filter((o) => o.name.includes(search))}
  onInputChange={(e) => setSearch(e.target.value)}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

**Đúng:**

```tsx
// ✅ Tắt filterBySearch khi tự lọc bên ngoài
<Select
  searchable
  filterBySearch={false}
  value={selected}
  onChange={setSelected}
  options={filteredOptions}
  onInputChange={(e) => handleSearch(e.target.value)}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

---

### RULE-SELECT-06: Sử dụng `clearable` để hiển thị nút xoá — không tự build clear button

Khi `clearable=true`, nút xoá (x) tự động hiển thị khi hover hoặc focus và component có giá trị. Click nút xoá sẽ gọi `onChange(null)`. Không tự build nút clear bên ngoài.

**Sai:**

```tsx
// ❌ Tự build nút clear bên ngoài
<div className={cx("flex items-center gap-[0.5rem]")}>
  <Select
    value={selected}
    onChange={setSelected}
    options={users}
    getLabel={(opt) => opt.name}
    getValue={(opt) => opt.id}
  />
  <button onClick={() => setSelected(null)}>Xoá</button>
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng clearable — component tự xử lý
<Select
  clearable
  value={selected}
  onChange={setSelected}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

---

### RULE-SELECT-07: Sử dụng `error` kết hợp `helperText` cho validation — không tự thêm HelperText bên ngoài

Component đã tích hợp sẵn `HelperText` bên dưới. Khi `error=true`, `helperText` tự động hiển thị với color error. Không tự render `HelperText` bên ngoài component.

**Sai:**

```tsx
// ❌ Tự thêm HelperText bên ngoài — bị duplicate
<div>
  <Select
    value={selected}
    onChange={setSelected}
    options={users}
    getLabel={(opt) => opt.name}
    getValue={(opt) => opt.id}
  />
  <HelperText color="error">Vui lòng chọn một giá trị</HelperText>
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng error và helperText của component
<Select
  error={hasError}
  helperText="Vui lòng chọn một giá trị"
  value={selected}
  onChange={setSelected}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

---

### RULE-SELECT-08: Sử dụng `renderOption` và `renderValue` để tuỳ chỉnh hiển thị — không override CSS nội bộ

`renderOption` tuỳ chỉnh hiển thị từng option trong dropdown. `renderValue` tuỳ chỉnh hiển thị giá trị đã chọn trên input. Không override CSS selector nội bộ để thay đổi giao diện.

**Sai:**

```tsx
// ❌ Override CSS nội bộ — dễ vỡ khi component cập nhật
<Select
  className={cx("[&_.stringee-select-option]:text-red-500")}
  value={selected}
  onChange={setSelected}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng renderOption và renderValue
<Select
  value={selected}
  onChange={setSelected}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
  renderOption={(opt) => (
    <div className={cx("flex items-center gap-[0.5rem]")}>
      <Avatar src={opt.avatar} />
      <span>{opt.name}</span>
    </div>
  )}
  renderValue={(opt) => (
    <div className={cx("flex items-center gap-[0.5rem]")}>
      <Avatar src={opt.avatar} />
      <span>{opt.name}</span>
    </div>
  )}
/>
```

---

### RULE-SELECT-09: `disabled` và `readOnly` có hành vi khác nhau — không nhầm lẫn

`disabled` làm component xám, không tương tác được. `readOnly` làm component trong suốt, vẫn hiển thị giá trị nhưng không mở dropdown. Cả hai đều vô hiệu hoá Popper. Dùng `readOnly` khi chỉ muốn hiển thị giá trị, dùng `disabled` khi field thực sự bị vô hiệu hoá.

**Sai:**

```tsx
// ❌ Dùng disabled khi chỉ muốn hiển thị giá trị — giao diện xám không đúng mục đích
<Select
  disabled
  value={selected}
  onChange={setSelected}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

**Đúng:**

```tsx
// ✅ Dùng readOnly để hiển thị giá trị mà không cho chỉnh sửa
<Select
  readOnly
  value={selected}
  onChange={setSelected}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>

// ✅ Dùng disabled khi field thực sự bị vô hiệu hoá
<Select
  disabled
  value={selected}
  onChange={setSelected}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

---

### RULE-SELECT-10: Sử dụng `disabledValues` để vô hiệu hoá option cụ thể — không tự lọc ra khỏi danh sách

`disabledValues` nhận mảng các value (trả về từ `getValue`) cần vô hiệu hoá. Option bị disable sẽ xám đi và không click được nhưng vẫn hiển thị trong danh sách. Không tự lọc option ra khỏi mảng `options`.

**Sai:**

```tsx
// ❌ Tự lọc option — người dùng không biết option tồn tại
<Select
  value={selected}
  onChange={setSelected}
  options={users.filter((u) => u.id !== currentUserId)}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng disabledValues — option vẫn hiển thị nhưng không chọn được
<Select
  value={selected}
  onChange={setSelected}
  options={users}
  disabledValues={[currentUserId]}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

---

### RULE-SELECT-11: Sử dụng `fullWidth` thay vì override className cho chiều rộng

Component kế thừa `fullWidth` từ `TextFieldProps`. Sử dụng `fullWidth` để component chiếm toàn bộ chiều rộng parent. Không tự override width qua `className`.

**Sai:**

```tsx
// ❌ Override width qua className — dễ conflict với logic nội bộ
<Select
  className={cx("w-full")}
  value={selected}
  onChange={setSelected}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng fullWidth
<Select
  fullWidth
  value={selected}
  onChange={setSelected}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

---

### RULE-SELECT-12: Sử dụng `onScrollToBottom` cho lazy loading — không tự build scroll listener

Component tích hợp sẵn `Waypoint` ở cuối danh sách option. Khi người dùng cuộn đến cuối, `onScrollToBottom` được gọi. Sử dụng prop này để tải thêm dữ liệu. Không tự build scroll listener bên ngoài.

**Sai:**

```tsx
// ❌ Tự build scroll listener — trùng lặp với logic có sẵn
<div onScroll={handleScroll}>
  <Select
    value={selected}
    onChange={setSelected}
    options={users}
    getLabel={(opt) => opt.name}
    getValue={(opt) => opt.id}
  />
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng onScrollToBottom
<Select
  value={selected}
  onChange={setSelected}
  options={users}
  onScrollToBottom={() => loadMore()}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

---

### RULE-SELECT-13: Sử dụng `hidePopperOnSelect={false}` khi không muốn đóng dropdown sau khi chọn

Mặc định `hidePopperOnSelect=true` — dropdown đóng khi chọn option. Khi cần giữ dropdown mở sau khi chọn (ví dụ: kết hợp với operator select), truyền `hidePopperOnSelect={false}`.

**Sai:**

```tsx
// ❌ Tự quản lý state mở/đóng dropdown — conflict với logic nội bộ
const [isOpen, setIsOpen] = useState(false);

<Select
  value={selected}
  onChange={(opt) => {
    setSelected(opt);
    setIsOpen(true); // Cố giữ dropdown mở nhưng không đúng cách
  }}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng hidePopperOnSelect={false}
<Select
  hidePopperOnSelect={false}
  value={selected}
  onChange={setSelected}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

---

### RULE-SELECT-14: Sử dụng `isEqual` khi cần so sánh option tuỳ chỉnh

Mặc định component dùng `lodash.isEqual` để so sánh value. Khi cần logic so sánh khác (ví dụ: chỉ so sánh theo id), truyền `isEqual`. Không tự viết logic so sánh bên ngoài để xác định option đang chọn.

**Sai:**

```tsx
// ❌ Tự viết logic so sánh bên ngoài — không đồng bộ với hiển thị check icon
<Select
  value={users.find((u) => u.id === selectedId) ?? null}
  onChange={(opt) => setSelectedId(opt?.id ?? null)}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng isEqual để so sánh theo id
<Select
  value={selected}
  onChange={setSelected}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
  isEqual={(val1, val2) => val1 === val2}
/>
```

---

### RULE-SELECT-15: Sử dụng `height` để thay đổi chiều cao input — không override CSS

Mặc định `height=40` (pixel). Truyền `height` để thay đổi chiều cao. Component sử dụng CSS variable `--height` nên tất cả logic nội bộ tự động điều chỉnh. Không override chiều cao bằng className.

**Sai:**

```tsx
// ❌ Override height bằng className — không đồng bộ với CSS variable nội bộ
<Select
  className={cx("[&_.stringee-input-root]:h-[2rem]")}
  value={selected}
  onChange={setSelected}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng prop height
<Select
  height={32}
  value={selected}
  onChange={setSelected}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
/>
```
