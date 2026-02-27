## Cách sử dụng component TreeSelect trong ui-kit

Path: `ui-kit/src/components/TreeSelect`

### Props

| Prop | Bắt buộc | Mặc định | Mô tả |
|---|---|---|---|
| `value` | ✓ | — | `Key[]` — keys đang chọn |
| `onChange` | ✓ | — | `(event: TreeSelectEvent<D>) => void` |
| `modeSelectTreeData` | ✓ | — | Chế độ chọn (xem bảng bên dưới) |
| `options` | ✓ | — | Dữ liệu cây |
| `selectedOptions` | — | — | Nodes đã chọn (khi options bị lọc/phân trang) |
| `clearable` | — | — | Nút xoá |
| `onClear` | — | — | Callback khi xoá |
| `showSearch` | — | — | Ô tìm kiếm trong dropdown |
| `searchInputProps` | — | — | Props cho ô tìm kiếm |
| `disableNodeIds` | — | — | `Key[]` — nodes bị disable |
| `maxLength` | — | `null` | Giới hạn số lượng chọn (không áp dụng MULTIPLE_ALL) |
| `usePoper` | — | `true` | `false` để nhúng tree trực tiếp |
| `showBorderOnBlur` | — | `true` | `false` để ẩn border khi blur |
| `inputProps` | — | — | Props cho input: `{ error, helperText, fullWidth, ... }` |
| `treeProps` | — | — | Props cho rc-tree: `{ fieldNames: { key, title, children } }` |
| `showOperatorSelect` | — | `false` | Select toán tử (AND/OR) |
| `operator` | — | — | Giá trị operator đang chọn |
| `operatorOptions` | — | — | `{ label, value, icon }[]` |
| `onChangeOperator` | — | — | Setter operator |
| `minWidthPopper` | — | `248` | Chiều rộng tối thiểu dropdown |

### modeSelectTreeData

| Mode | Chọn | Node | Icon |
|---|---|---|---|
| `SINGLE_CHILD` | 1 | Chỉ lá | Radio |
| `SINGLE_ANY` | 1 | Bất kỳ | Radio |
| `SINGLE_ANY_NO_ICON` | 1 | Bất kỳ | Không |
| `MULTIPLE_CHILD` | Nhiều | Chỉ lá | Checkbox |
| `MULTIPLE_ANY` | Nhiều | Bất kỳ | Checkbox |
| `MULTIPLE_ALL` | Nhiều | Chọn cha → chọn hết con | Checkbox |

### Basic usage

```tsx
const [selectedKeys, setSelectedKeys] = useState<Key[]>([]);

<TreeSelect
  value={selectedKeys}
  onChange={(e) => setSelectedKeys(e.selectedKeys)}
  options={departments}
  modeSelectTreeData="MULTIPLE_CHILD"
  treeProps={{ fieldNames: { key: "id", title: "name", children: "subDepts" } }}
  showSearch
  clearable
  onClear={() => setSelectedKeys([])}
  inputProps={{ fullWidth: true }}
/>
```

### Với operator select

```tsx
<TreeSelect
  showOperatorSelect
  operatorOptions={[
    { label: "Tất cả", value: "and" },
    { label: "Bất kỳ", value: "or" },
  ]}
  operator={operator}
  onChangeOperator={setOperator}
  value={selectedKeys}
  onChange={(e) => setSelectedKeys(e.selectedKeys)}
  options={treeData}
  modeSelectTreeData="MULTIPLE_ANY"
/>
```

### Nhúng tree trực tiếp (không Popper)

```tsx
<TreeSelect
  usePoper={false}
  value={selectedKeys}
  onChange={(e) => setSelectedKeys(e.selectedKeys)}
  options={treeData}
  modeSelectTreeData="MULTIPLE_ANY"
/>
```

### DO / DON'T

DO: Dùng `treeProps.fieldNames` để ánh xạ dữ liệu — không tự map sang `{ key, title, children }`.

DO: Dùng `maxLength` thay vì check trong `onChange`.

DO: Dùng `disableNodeIds` thay vì lọc options — node vẫn hiển thị nhưng không chọn được.

DO: Truyền `selectedOptions` khi options bị lọc/phân trang — tránh tag hiển thị key thay vì label.

DO: Dùng `inputProps={{ error, helperText }}` — không tự render HelperText bên ngoài.

DON'T: Tự viết logic giới hạn chọn trong `onChange` — dùng `modeSelectTreeData` đúng loại.

DON'T: Tự build ô tìm kiếm bên ngoài — dùng `showSearch`.

DON'T: Tự import `rc-tree` — dùng `usePoper={false}` để nhúng tree trực tiếp.

DON'T: Override CSS border — dùng `showBorderOnBlur={false}`.
