---
title: Cách sử dụng MultipleSelect Component của ui-kit
impact: HIGH
impactDescription: Sử dụng MultipleSelect sai dẫn đến việc truyền props không đúng, xử lý giá trị selected sai, hiển thị tag không nhất quán và lỗi logic khi dùng chế độ singleSelect hoặc changeOnBlur
tags: multiple-select, select, multiselect, dropdown, tag, checkbox, search, ui-kit
---

## Cách sử dụng MultipleSelect Component của ui-kit

Đường dẫn component: `ui-kit/src/components/MultipleSelect`

Props bắt buộc: `value`, `options`, `onChange`, `getLabel`, `getValue`.

Props thường dùng: `placeholder`, `searchable`, `clearable`, `disabled`, `readOnly`, `fullWidth`, `error`, `helperText`, `singleSelect`, `hasSelectAll`, `maxSelectedValue`, `disabledValues`, `showCheckbox`, `showCheckIcon`, `tagVariant`, `changeOnBlur`, `hideAfterSelect`, `renderOption`, `renderTag`, `renderLimitTag`, `popperProps`, `filterBySearch`, `onInputChange`, `searchValue`, `startAdornment`, `isShowIconSearch`, `renderEndAction`, `valueTooltip`.

Component là **generic** `MultipleSelect<T>`, kiểu `T` được suy ra từ `options`.

---

### RULE-MSEL-01: Luôn truyền đủ 5 props bắt buộc: `value`, `options`, `onChange`, `getLabel`, `getValue`

`MultipleSelect` yêu cầu 5 props bắt buộc. `value` là mảng các phần tử đã chọn, `options` là mảng tất cả các lựa chọn, `onChange` nhận mảng mới, `getLabel` trả về label hiển thị, `getValue` trả về giá trị duy nhất để so sánh.

**Sai:**

```tsx
// ❌ Thiếu getLabel và getValue — component không biết cách hiển thị và so sánh option
<MultipleSelect
  value={selectedUsers}
  options={users}
  onChange={setSelectedUsers}
/>
```

**Đúng:**

```tsx
// ✅ Truyền đầy đủ 5 props bắt buộc
<MultipleSelect
  value={selectedUsers}
  options={users}
  onChange={setSelectedUsers}
  getLabel={(user) => user.name}
  getValue={(user) => user.id}
/>
```

---

### RULE-MSEL-02: Sử dụng `singleSelect` thay vì tự xây dựng logic chọn một

Khi chỉ cần chọn một giá trị nhưng vẫn muốn hiển thị dạng tag và có search, sử dụng prop `singleSelect={true}`. Khi bật `singleSelect`, tag sẽ không hiển thị nút xoá, chọn option mới sẽ thay thế option cũ, và checkbox sẽ chuyển thành radio button.

**Sai:**

```tsx
// ❌ Tự giới hạn selection bằng logic onChange — mất nhiều hành vi mặc định
<MultipleSelect
  value={selected}
  options={items}
  onChange={(values) => setSelected(values.slice(-1))}
  getLabel={(item) => item.name}
  getValue={(item) => item.id}
/>
```

**Đúng:**

```tsx
// ✅ Dùng singleSelect để component tự xử lý logic chọn một
<MultipleSelect
  singleSelect
  value={selected}
  options={items}
  onChange={setSelected}
  getLabel={(item) => item.name}
  getValue={(item) => item.id}
/>
```

---

### RULE-MSEL-03: Sử dụng `changeOnBlur` khi muốn trì hoãn thay đổi giá trị

Khi bật `changeOnBlur={true}`, `onChange` chỉ được gọi khi người dùng click ra ngoài hoặc nhấn Tab (không gọi mỗi lần chọn/bỏ chọn option). Nhấn Escape sẽ huỷ bỏ thay đổi và khôi phục giá trị ban đầu. Thích hợp cho các trường hợp cần xác nhận trước khi lưu.

**Sai:**

```tsx
// ❌ Tự quản lý state tạm rồi đồng bộ khi blur — logic dư thừa
const [tempValue, setTempValue] = useState(value);
<MultipleSelect
  value={tempValue}
  onChange={setTempValue}
  onBlur={() => onSave(tempValue)}
  ...
/>
```

**Đúng:**

```tsx
// ✅ changeOnBlur tự quản lý state tạm và chỉ gọi onChange khi đóng dropdown
<MultipleSelect
  changeOnBlur
  value={selectedItems}
  onChange={handleSave}
  options={items}
  getLabel={(item) => item.name}
  getValue={(item) => item.id}
/>
```

---

### RULE-MSEL-04: Sử dụng `disabledValues` thay vì lọc options để vô hiệu hoá một số lựa chọn

`disabledValues` nhận mảng giá trị (kết quả của `getValue`) hoặc một hàm filter `(option: T) => boolean`. Các option bị disabled vẫn hiển thị nhưng không thể chọn/bỏ chọn. Kết hợp với `tooltipOfDisabledOption` để giải thích lý do.

**Sai:**

```tsx
// ❌ Lọc bỏ option — người dùng không thấy và không hiểu tại sao thiếu lựa chọn
<MultipleSelect
  options={users.filter((u) => !u.isLocked)}
  ...
/>
```

**Đúng:**

```tsx
// ✅ Dùng disabledValues với mảng giá trị
<MultipleSelect
  options={users}
  disabledValues={lockedUserIds}
  tooltipOfDisabledOption={(user) => `${user.name} đang bị khoá`}
  ...
/>

// ✅ Hoặc dùng disabledValues với hàm filter
<MultipleSelect
  options={users}
  disabledValues={(user) => user.isLocked}
  tooltipOfDisabledOption={(user) => `${user.name} đang bị khoá`}
  ...
/>
```

---

### RULE-MSEL-05: Sử dụng `clearable` và `alsoClearDisabledValues` đúng cách

Khi `clearable={true}`, nút xoá tất cả sẽ hiển thị. Mặc định, nút xoá sẽ giữ lại các giá trị bị disabled. Nếu muốn xoá luôn cả giá trị bị disabled, truyền thêm `alsoClearDisabledValues={true}`.

**Sai:**

```tsx
// ❌ Tự tạo nút clear bên ngoài — trùng lặp chức năng
<div className={cx("relative")}>
  <MultipleSelect value={selected} onChange={setSelected} ... />
  <button onClick={() => setSelected([])}>X</button>
</div>
```

**Đúng:**

```tsx
// ✅ Dùng clearable — nút clear hiển thị khi có giá trị được chọn
<MultipleSelect
  clearable
  value={selected}
  onChange={setSelected}
  ...
/>

// ✅ Xoá tất cả bao gồm cả giá trị bị disabled
<MultipleSelect
  clearable
  alsoClearDisabledValues
  disabledValues={lockedIds}
  value={selected}
  onChange={setSelected}
  ...
/>
```

---

### RULE-MSEL-06: Sử dụng `searchValue` và `onInputChange` cho tìm kiếm từ server (remote search)

Khi cần tìm kiếm từ API (remote search), truyền `searchValue` để component nhận biết đang ở chế độ remote search. Kết hợp với `onInputChange` để gọi API và `filterBySearch={false}` để tắt lọc phía client.

**Sai:**

```tsx
// ❌ Tự quản lý search state bên ngoài mà không dùng searchValue — component vẫn filter local
<MultipleSelect
  options={remoteOptions}
  onInputChange={(e) => fetchOptions(e.target.value)}
  ...
/>
```

**Đúng:**

```tsx
// ✅ Dùng searchValue + filterBySearch={false} cho remote search
const [searchText, setSearchText] = useState("");

<MultipleSelect
  searchValue={searchText}
  filterBySearch={false}
  onInputChange={(e) => {
    setSearchText(e.target.value);
    debouncedFetch(e.target.value);
  }}
  options={remoteOptions}
  optionListLoading={isLoading}
  ...
/>
```

---

### RULE-MSEL-07: Sử dụng `renderOption` để tuỳ chỉnh hiển thị option, không override `getLabel`

`getLabel` dùng để lấy text label (phục vụ tìm kiếm, tooltip, tag). `renderOption` dùng để tuỳ chỉnh giao diện hiển thị trong danh sách dropdown. Không nên trả về JSX phức tạp từ `getLabel`.

**Sai:**

```tsx
// ❌ getLabel trả về JSX phức tạp — ảnh hưởng đến tìm kiếm và hiển thị tag
<MultipleSelect
  getLabel={(user) => (
    <div className={cx("flex items-center gap-[0.5rem]")}>
      <Avatar src={user.avatar} />
      <span>{user.name}</span>
    </div>
  )}
  ...
/>
```

**Đúng:**

```tsx
// ✅ getLabel trả về text thuần, renderOption tuỳ chỉnh giao diện dropdown
<MultipleSelect
  getLabel={(user) => user.name}
  renderOption={(user) => (
    <div className={cx("flex items-center gap-[0.5rem]")}>
      <Avatar src={user.avatar} />
      <span>{user.name}</span>
    </div>
  )}
  ...
/>
```

---

### RULE-MSEL-08: Sử dụng `renderTag` để tuỳ chỉnh hiển thị tag, không wrap bên ngoài

`renderTag` nhận `RenderTagParams<T>` gồm: `option`, `index`, `selectedOptions`, `disabled`, `displayValueNumber`, `onDelete`, `isFocusedInput`. Sử dụng các params này để render tag tuỳ chỉnh.

**Sai:**

```tsx
// ❌ Tự render tag list bên ngoài — mất tính năng limit tag, keyboard navigation
<div>
  {selected.map((item) => (
    <CustomTag key={item.id}>{item.name}</CustomTag>
  ))}
  <MultipleSelect value={selected} ... />
</div>
```

**Đúng:**

```tsx
// ✅ Dùng renderTag để tuỳ chỉnh từng tag
<MultipleSelect
  renderTag={({ option, onDelete, disabled }) => (
    <Tag
      variant="outlined"
      color={option.color}
      onDelete={!disabled ? onDelete : undefined}
    >
      {option.name}
    </Tag>
  )}
  ...
/>
```

---

### RULE-MSEL-09: Sử dụng `hideAfterSelect` khi muốn đóng dropdown sau mỗi lần chọn

Mặc định, dropdown giữ mở để người dùng chọn nhiều option. Khi cần đóng sau mỗi lần chọn (thường kết hợp với `singleSelect`), truyền `hideAfterSelect={true}`.

**Sai:**

```tsx
// ❌ Tự đóng dropdown bằng ref — phức tạp và dễ lỗi
const popperRef = useRef();
<MultipleSelect
  singleSelect
  onChange={(values) => {
    setSelected(values);
    popperRef.current?.hide();
  }}
  ...
/>
```

**Đúng:**

```tsx
// ✅ Dùng hideAfterSelect — dropdown tự đóng sau khi chọn
<MultipleSelect
  singleSelect
  hideAfterSelect
  value={selected}
  onChange={setSelected}
  ...
/>
```

---

### RULE-MSEL-10: Sử dụng `maxSelectedValue` để giới hạn số lượng lựa chọn

Khi cần giới hạn số option được chọn, truyền `maxSelectedValue`. Các option chưa được chọn sẽ bị disabled khi đạt giới hạn. Các option đã chọn vẫn có thể bỏ chọn.

**Sai:**

```tsx
// ❌ Tự kiểm tra trong onChange — option vẫn hiển thị clickable gây nhầm lẫn cho người dùng
<MultipleSelect
  onChange={(values) => {
    if (values.length > 5) return;
    setSelected(values);
  }}
  ...
/>
```

**Đúng:**

```tsx
// ✅ maxSelectedValue tự động disable các option chưa chọn khi đạt giới hạn
<MultipleSelect
  maxSelectedValue={5}
  value={selected}
  onChange={setSelected}
  ...
/>
```

---

### RULE-MSEL-11: Sử dụng `hasSelectAll` để hiển thị option "Chọn tất cả"

Khi bật `hasSelectAll={true}`, một option "Tất cả" sẽ hiển thị ở đầu danh sách với checkbox. Click vào sẽ chọn tất cả hoặc bỏ chọn tất cả. Không hoạt động khi `singleSelect={true}`.

**Sai:**

```tsx
// ❌ Tự thêm option "Chọn tất cả" vào mảng options — gây lỗi logic getValue/getLabel
const optionsWithAll = [{ id: "all", name: "Chọn tất cả" }, ...users];
<MultipleSelect options={optionsWithAll} ... />
```

**Đúng:**

```tsx
// ✅ Dùng hasSelectAll — component tự xử lý logic chọn/bỏ chọn tất cả
<MultipleSelect
  hasSelectAll
  value={selected}
  options={users}
  onChange={setSelected}
  getLabel={(user) => user.name}
  getValue={(user) => user.id}
/>
```

---

### RULE-MSEL-12: Sử dụng `isShowIconSearch` khi muốn hiển thị icon search thay vì nút clear

Khi bật `isShowIconSearch={true}`, icon search sẽ hiển thị ở bên phải. Khi đã có giá trị được chọn, click vào container sẽ không mở dropdown nữa mà phải click vào icon search. Mỗi tag sẽ luôn hiển thị nút xoá (không cần focus). Prop `clearable` sẽ bị ẩn khi `isShowIconSearch` được bật.

```tsx
// ✅ Hiển thị icon search — phù hợp cho trường hợp cần thao tác xoá tag riêng lẻ
<MultipleSelect
  isShowIconSearch
  value={selected}
  options={users}
  onChange={setSelected}
  getLabel={(user) => user.name}
  getValue={(user) => user.id}
/>
```

---

### RULE-MSEL-13: Sử dụng `onScrollToBottom` cho lazy loading danh sách option

Khi danh sách option rất dài và cần phân trang, sử dụng `onScrollToBottom` để load thêm option khi cuộn đến cuối. Kết hợp với `optionListLoading` để hiển thị trạng thái loading và `showMoreText` để hiển thị thông tin bổ sung ở cuối danh sách.

```tsx
// ✅ Lazy loading options
<MultipleSelect
  options={users}
  optionListLoading={isFetchingMore}
  onScrollToBottom={() => fetchNextPage()}
  showMoreText={hasMore ? "Đang tải thêm..." : null}
  ...
/>
```

---

### RULE-MSEL-14: Không truyền rõ ràng các giá trị mặc định

Các props sau đã có giá trị mặc định, chỉ truyền khi cần thay đổi:

| Prop | Giá trị mặc định |
|------|-------------------|
| `searchable` | `true` |
| `showCheckbox` | `true` |
| `showCheckIcon` | `true` |
| `showBorderOnBlur` | `true` |
| `showTooltip` | `true` |
| `filterBySearch` | `true` |
| `appendToBody` | `true` |
| `tagVariant` | `"contained"` |
| `disabled` | `false` |
| `error` | `false` |
| `fullWidth` | `false` |
| `clearable` | `false` |
| `singleSelect` | `false` |
| `changeOnBlur` | `false` |
| `hideAfterSelect` | `false` |
| `readOnly` | `false` |
| `tabIndex` | `0` |

**Sai:**

```tsx
// ❌ Truyền lại giá trị mặc định — dư thừa
<MultipleSelect
  searchable={true}
  showCheckbox={true}
  disabled={false}
  clearable={false}
  appendToBody={true}
  ...
/>
```

**Đúng:**

```tsx
// ✅ Chỉ truyền khi cần thay đổi giá trị mặc định
<MultipleSelect
  clearable
  fullWidth
  ...
/>
```

---

### RULE-MSEL-15: Sử dụng `renderLimitTag` và `renderLimitValue` để tuỳ chỉnh hiển thị tag bị giới hạn

Khi số lượng tag vượt quá chiều rộng container, các tag thừa sẽ được ẩn và hiển thị dạng `+N`. Click vào `+N` sẽ hiển thị popper chứa danh sách các giá trị bị ẩn.

- `renderLimitTag`: tuỳ chỉnh hiển thị của badge `+N`
- `renderLimitValue`: tuỳ chỉnh hiển thị từng item trong popper danh sách giá trị bị ẩn

```tsx
// ✅ Tuỳ chỉnh limit tag badge và nội dung popper
<MultipleSelect
  renderLimitTag={(tailNumber) => (
    <Tag variant="outlined">và {tailNumber} người khác</Tag>
  )}
  renderLimitValue={(user) => (
    <div className={cx("flex items-center gap-[0.5rem]")}>
      <Avatar src={user.avatar} size="small" />
      <span>{user.name}</span>
    </div>
  )}
  ...
/>
```

---

### RULE-MSEL-16: Sử dụng `startAdornment` và `alwaysShowAdornment` để thêm icon/text phía trước

`startAdornment` hiển thị phần tử ở đầu input. Mặc định chỉ hiển thị khi chưa có giá trị nào được chọn. Nếu muốn luôn hiển thị, truyền thêm `alwaysShowAdornment={true}`.

```tsx
// ✅ Hiển thị icon chỉ khi chưa chọn giá trị
<MultipleSelect
  startAdornment={<FontAwesomeIcon icon={faUser} />}
  placeholder="Chọn nhân viên"
  ...
/>

// ✅ Luôn hiển thị icon kể cả khi đã chọn giá trị
<MultipleSelect
  startAdornment={<FontAwesomeIcon icon={faUser} />}
  alwaysShowAdornment
  ...
/>
```

---

## Tham chiếu style

| Trạng thái | Border | Background |
|------------|--------|------------|
| Mặc định | `border-divider-primary` | `bg-primary-light-98` |
| Hover | `border-primary-light-70` | `bg-primary-light-96` |
| Focus (mở dropdown) | `border-info` | `bg-primary-light-96` |
| Disabled | `border-primary-light-94` | `bg-primary-light-94` |
| Error | `border-error` | — |
| ReadOnly | `border-transparent` | `bg-transparent` |

| Phần tử | Kích thước |
|---------|-----------|
| Container min-height | `40px` |
| Container max-height | `100px` |
| Container width mặc định | `190px` |
| Option padding | `px-16px py-6px` |
| Search input padding | `py-8px pl-44px pr-16px` |
