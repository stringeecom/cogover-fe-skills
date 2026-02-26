---
title: Cách sử dụng component TreeSelect trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến mất dữ liệu chọn, dropdown không hoạt động, lỗi chế độ chọn (single/multiple), và hành vi chọn node không nhất quán
tags: tree-select, tree, select, dropdown, hierarchical, rc-tree, ui-kit
---

## Cách sử dụng component TreeSelect trong ui-kit

Đường dẫn component: `ui-kit/src/components/TreeSelect`

Props bắt buộc: `value`, `onChange`, `modeSelectTreeData`, `options`.
Props tùy chọn: `selectedOptions`, `placeholder`, `clearable`, `clearIcon`, `onClear`, `showSearch`, `searchInputProps`, `disabled`, `height`, `maxLength`, `usePoper`, `disableNodeIds`, `showOperatorSelect`, `operatorOptions`, `onChangeOperator`, `operator`, `showBorderOnBlur`, `tabIndex`, `textNoOptions`, `minWidthPopper`, `popperProps`, `inputProps`, `treeProps`.

Kiểu dữ liệu: `TreeSelectEvent<D> = { selectedKeys: Key[]; event?: EventDataNode<D> }`.

Chế độ chọn (`modeSelectTreeData`): `SINGLE_CHILD` (chọn 1, chỉ node lá), `SINGLE_ANY` (chọn 1, bất kỳ node nào), `SINGLE_ANY_NO_ICON` (chọn 1, bất kỳ node nào, không hiển thị Radio), `MULTIPLE_CHILD` (chọn nhiều, chỉ node lá), `MULTIPLE_ANY` (chọn nhiều, bất kỳ node nào), `MULTIPLE_ALL` (chọn nhiều, chọn node cha sẽ chọn tất cả node con).

Giá trị mặc định: `usePoper=true`, `showBorderOnBlur=true`, `showOperatorSelect=false`, `tabIndex=0`, `minWidthPopper=248`, `maxLength=null`.

---

### RULE-TREE-SELECT-01: Luôn truyền đủ 4 props bắt buộc — `value`, `onChange`, `modeSelectTreeData`, `options`

`TreeSelect` là controlled component. Phải truyền đủ: `value` (mảng Key đang chọn), `onChange` (callback nhận `TreeSelectEvent`), `modeSelectTreeData` (chế độ chọn), `options` (dữ liệu dạng cây).

**Sai:**

```tsx
// ❌ Thiếu modeSelectTreeData — component không biết chế độ chọn
<TreeSelect
  value={selectedKeys}
  onChange={handleChange}
  options={treeData}
/>
```

**Đúng:**

```tsx
// ✅
<TreeSelect
  value={selectedKeys}
  onChange={handleChange}
  options={treeData}
  modeSelectTreeData="SINGLE_ANY"
/>
```

---

### RULE-TREE-SELECT-02: Sử dụng `treeProps.fieldNames` để ánh xạ dữ liệu — không tự chuyển đổi dữ liệu

Component sử dụng `treeProps.fieldNames` với 3 trường `key`, `title`, `children` (mặc định lần lượt là `"key"`, `"title"`, `"children"`). Khi dữ liệu có tên trường khác, truyền `fieldNames` thay vì tự map lại dữ liệu.

**Sai:**

```tsx
// ❌ Tự map dữ liệu sang cấu trúc mới — tốn bộ nhớ và mất tham chiếu gốc
const mappedData = departments.map((d) => ({
  key: d.id,
  title: d.name,
  children: d.subDepts?.map((s) => ({ key: s.id, title: s.name })),
}));

<TreeSelect
  value={selectedKeys}
  onChange={handleChange}
  options={mappedData}
  modeSelectTreeData="SINGLE_ANY"
/>
```

**Đúng:**

```tsx
// ✅ Dùng fieldNames để ánh xạ trực tiếp
<TreeSelect
  value={selectedKeys}
  onChange={handleChange}
  options={departments}
  modeSelectTreeData="SINGLE_ANY"
  treeProps={{
    fieldNames: { key: "id", title: "name", children: "subDepts" },
  }}
/>
```

---

### RULE-TREE-SELECT-03: Chọn đúng `modeSelectTreeData` theo yêu cầu nghiệp vụ — không tự viết logic chọn

Component hỗ trợ 6 chế độ chọn. Mỗi chế độ tự xử lý logic chọn node (Radio/Checkbox, node lá/bất kỳ, chọn nhóm). Không tự viết logic giới hạn hoặc mở rộng phạm vi chọn.

- `SINGLE_CHILD`: Chọn 1, chỉ node lá, hiển thị Radio.
- `SINGLE_ANY`: Chọn 1, bất kỳ node nào, hiển thị Radio.
- `SINGLE_ANY_NO_ICON`: Chọn 1, bất kỳ node nào, không hiển thị Radio — node được chọn chỉ đổi nền.
- `MULTIPLE_CHILD`: Chọn nhiều, chỉ node lá, hiển thị Checkbox.
- `MULTIPLE_ANY`: Chọn nhiều, bất kỳ node nào, hiển thị Checkbox.
- `MULTIPLE_ALL`: Chọn nhiều, chọn node cha sẽ tự động chọn tất cả node con, hiển thị Checkbox.

**Sai:**

```tsx
// ❌ Dùng MULTIPLE_ANY rồi tự lọc chỉ giữ node lá — logic thừa
<TreeSelect
  value={selectedKeys}
  onChange={(e) => {
    const leafOnly = e.selectedKeys.filter((k) => isLeafNode(k));
    setSelectedKeys(leafOnly);
  }}
  options={treeData}
  modeSelectTreeData="MULTIPLE_ANY"
/>
```

**Đúng:**

```tsx
// ✅ Dùng MULTIPLE_CHILD — component tự giới hạn chỉ chọn node lá
<TreeSelect
  value={selectedKeys}
  onChange={(e) => setSelectedKeys(e.selectedKeys)}
  options={treeData}
  modeSelectTreeData="MULTIPLE_CHILD"
/>
```

---

### RULE-TREE-SELECT-04: Sử dụng `clearable` và `onClear` để xoá giá trị — không tự build nút clear

Khi `clearable=true`, nút xoá (×) tự hiển thị khi hover và component có giá trị. Click nút xoá gọi `onChange({ selectedKeys: [] })` và `onClear`. Không tự build nút clear bên ngoài.

**Sai:**

```tsx
// ❌ Tự build nút clear bên ngoài — mất đồng bộ state
<div className={cx("flex items-center gap-[0.5rem]")}>
  <TreeSelect
    value={selectedKeys}
    onChange={handleChange}
    options={treeData}
    modeSelectTreeData="SINGLE_ANY"
  />
  <button onClick={() => setSelectedKeys([])}>Xoá</button>
</div>
```

**Đúng:**

```tsx
// ✅
<TreeSelect
  clearable
  onClear={() => console.log("123: Cleared")}
  value={selectedKeys}
  onChange={handleChange}
  options={treeData}
  modeSelectTreeData="SINGLE_ANY"
/>
```

---

### RULE-TREE-SELECT-05: Sử dụng `showSearch` để bật tìm kiếm — không tự build ô tìm kiếm bên ngoài

Khi `showSearch=true`, ô tìm kiếm hiển thị trong dropdown với auto focus. Component tự lọc tree theo từ khoá và tự mở rộng các node cha chứa kết quả. Dùng `searchInputProps` để tuỳ chỉnh ô tìm kiếm.

**Sai:**

```tsx
// ❌ Tự build ô tìm kiếm và lọc dữ liệu bên ngoài
const [search, setSearch] = useState("");
const filtered = filterByKeyword(treeData, search);

<TextField value={search} onChange={(e) => setSearch(e.target.value)} />
<TreeSelect
  value={selectedKeys}
  onChange={handleChange}
  options={filtered}
  modeSelectTreeData="MULTIPLE_ANY"
/>
```

**Đúng:**

```tsx
// ✅ Component tự xử lý tìm kiếm và mở rộng node
<TreeSelect
  showSearch
  value={selectedKeys}
  onChange={handleChange}
  options={treeData}
  modeSelectTreeData="MULTIPLE_ANY"
/>
```

---

### RULE-TREE-SELECT-06: Sử dụng `inputProps` cho validation — không tự thêm HelperText bên ngoài

Component đã tích hợp sẵn `HelperText` bên dưới. Khi `inputProps.error=true`, border chuyển đỏ và `inputProps.helperText` tự hiển thị với color error. Không tự render `HelperText` bên ngoài component.

**Sai:**

```tsx
// ❌ Tự thêm HelperText bên ngoài — bị duplicate
<div>
  <TreeSelect
    value={selectedKeys}
    onChange={handleChange}
    options={treeData}
    modeSelectTreeData="SINGLE_ANY"
  />
  <HelperText color="error">Vui lòng chọn ít nhất 1 mục</HelperText>
</div>
```

**Đúng:**

```tsx
// ✅
<TreeSelect
  value={selectedKeys}
  onChange={handleChange}
  options={treeData}
  modeSelectTreeData="SINGLE_ANY"
  inputProps={{
    error: hasError,
    helperText: "Vui lòng chọn ít nhất 1 mục",
  }}
/>
```

---

### RULE-TREE-SELECT-07: Sử dụng `maxLength` để giới hạn số lượng chọn — không tự giới hạn trong `onChange`

Prop `maxLength` tự giới hạn số lượng node được chọn ở chế độ multiple (trừ `MULTIPLE_ALL`). Khi đạt giới hạn, click thêm node sẽ không có tác dụng. Không tự viết logic giới hạn trong callback `onChange`.

**Sai:**

```tsx
// ❌ Tự giới hạn số lượng chọn — logic thừa
<TreeSelect
  value={selectedKeys}
  onChange={(e) => {
    if (e.selectedKeys.length <= 5) {
      setSelectedKeys(e.selectedKeys);
    }
  }}
  options={treeData}
  modeSelectTreeData="MULTIPLE_ANY"
/>
```

**Đúng:**

```tsx
// ✅ Component tự xử lý giới hạn
<TreeSelect
  maxLength={5}
  value={selectedKeys}
  onChange={(e) => setSelectedKeys(e.selectedKeys)}
  options={treeData}
  modeSelectTreeData="MULTIPLE_ANY"
/>
```

---

### RULE-TREE-SELECT-08: Sử dụng `disableNodeIds` để vô hiệu hoá node cụ thể — không lọc bỏ khỏi `options`

Prop `disableNodeIds` nhận mảng Key của các node cần vô hiệu hoá. Node bị disable vẫn hiển thị (text xám) nhưng không click được. Không lọc bỏ node khỏi `options` khi chỉ muốn vô hiệu hoá.

**Sai:**

```tsx
// ❌ Lọc bỏ node khỏi options — mất cấu trúc cây, node con cũng biến mất
const filteredOptions = treeData.filter((n) => !disabledIds.includes(n.id));

<TreeSelect
  value={selectedKeys}
  onChange={handleChange}
  options={filteredOptions}
  modeSelectTreeData="MULTIPLE_ANY"
  treeProps={{ fieldNames: { key: "id", title: "name" } }}
/>
```

**Đúng:**

```tsx
// ✅ Node vẫn hiển thị nhưng không chọn được
<TreeSelect
  value={selectedKeys}
  onChange={handleChange}
  options={treeData}
  modeSelectTreeData="MULTIPLE_ANY"
  disableNodeIds={disabledIds}
  treeProps={{ fieldNames: { key: "id", title: "name" } }}
/>
```

---

### RULE-TREE-SELECT-09: Sử dụng `usePoper={false}` khi cần nhúng tree trực tiếp — không tự render `rc-tree`

Khi `usePoper=false`, component render tree trực tiếp không có Popper/dropdown. Dùng cho trường hợp nhúng tree vào layout cố định. Không tự import `rc-tree` để render riêng.

**Sai:**

```tsx
// ❌ Tự import rc-tree — mất logic chọn, disable, chế độ chọn của TreeSelect
import Tree from "rc-tree";

<Tree
  treeData={treeData}
  selectedKeys={selectedKeys}
  onSelect={(keys) => setSelectedKeys(keys)}
/>
```

**Đúng:**

```tsx
// ✅ Giữ nguyên logic chọn của TreeSelect, chỉ bỏ Popper
<TreeSelect
  usePoper={false}
  value={selectedKeys}
  onChange={handleChange}
  options={treeData}
  modeSelectTreeData="MULTIPLE_ANY"
/>
```

---

### RULE-TREE-SELECT-10: Truyền `selectedOptions` khi giá trị đã chọn có thể không nằm trong `options` hiện tại

Khi `options` được lọc hoặc phân trang, các node đã chọn có thể không có trong `options`. Component dùng `selectedOptions` kết hợp `options` để tra cứu label hiển thị trên tag. Không truyền sẽ hiển thị key thay vì label.

**Sai:**

```tsx
// ❌ Không truyền selectedOptions — tag hiển thị key thay vì label
<TreeSelect
  value={selectedKeys}
  onChange={handleChange}
  options={filteredTreeData}
  modeSelectTreeData="MULTIPLE_ANY"
/>
```

**Đúng:**

```tsx
// ✅ Truyền selectedOptions để component tra cứu label cho các node đã chọn
<TreeSelect
  value={selectedKeys}
  onChange={handleChange}
  options={filteredTreeData}
  selectedOptions={allSelectedNodes}
  modeSelectTreeData="MULTIPLE_ANY"
/>
```

---

### RULE-TREE-SELECT-11: Sử dụng `showOperatorSelect` cho bộ lọc toán tử — không tự build select bên ngoài

Khi cần toán tử lọc (AND/OR), sử dụng `showOperatorSelect=true` kết hợp `operatorOptions`, `operator`, `onChangeOperator`. Component tự render Select toán tử phía trên tree. Không tự build Select bên ngoài.

**Sai:**

```tsx
// ❌ Tự build Select toán tử bên ngoài — mất vị trí đúng trong dropdown
<Select
  options={[{ label: "AND", value: "and" }, { label: "OR", value: "or" }]}
  value={operator}
  onChange={setOperator}
/>
<TreeSelect
  value={selectedKeys}
  onChange={handleChange}
  options={treeData}
  modeSelectTreeData="MULTIPLE_ANY"
/>
```

**Đúng:**

```tsx
// ✅ Component tự render Select toán tử bên trong dropdown
<TreeSelect
  showOperatorSelect
  operatorOptions={[
    { label: "Tất cả", value: "and", icon: <IconAnd /> },
    { label: "Bất kỳ", value: "or", icon: <IconOr /> },
  ]}
  operator={operator}
  onChangeOperator={setOperator}
  value={selectedKeys}
  onChange={handleChange}
  options={treeData}
  modeSelectTreeData="MULTIPLE_ANY"
/>
```

---

### RULE-TREE-SELECT-12: Sử dụng `showBorderOnBlur={false}` để ẩn border khi blur — không override CSS

Mặc định `showBorderOnBlur=true`, component hiển thị border khi không focus. Khi `showBorderOnBlur={false}`, border ẩn khi blur và chỉ hiện khi focus. Không override CSS selector nội bộ để ẩn border.

**Sai:**

```tsx
// ❌ Override CSS nội bộ — dễ vỡ khi component cập nhật
<TreeSelect
  inputProps={{
    className: cx("[&_.stringee-ts-select-input]:border-0"),
  }}
  value={selectedKeys}
  onChange={handleChange}
  options={treeData}
  modeSelectTreeData="SINGLE_ANY"
/>
```

**Đúng:**

```tsx
// ✅
<TreeSelect
  showBorderOnBlur={false}
  value={selectedKeys}
  onChange={handleChange}
  options={treeData}
  modeSelectTreeData="SINGLE_ANY"
/>
```
