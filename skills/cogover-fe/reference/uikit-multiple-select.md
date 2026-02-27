## Cách sử dụng MultipleSelect Component của ui-kit

Path: `ui-kit/src/components/MultipleSelect`
Generic: `MultipleSelect<T>` — kiểu `T` suy ra từ `options`.

### Props bắt buộc

| Prop | Mô tả |
|---|---|
| `value` | `T[]` — mảng items đã chọn |
| `options` | `T[]` — tất cả lựa chọn |
| `onChange` | `(items: T[]) => void` |
| `getLabel` | `(item: T) => string` — text thuần (dùng cho search, tag, tooltip) |
| `getValue` | `(item: T) => unknown` — unique identifier |

### Props thường dùng

| Prop | Mặc định | Mô tả |
|---|---|---|
| `searchable` | `true` | Tìm kiếm |
| `filterBySearch` | `true` | `false` khi remote search |
| `searchValue` | — | Kết hợp với `filterBySearch={false}` cho remote search |
| `onInputChange` | — | `(e) => void` |
| `clearable` | `false` | Nút xoá tất cả |
| `alsoClearDisabledValues` | — | Xoá cả disabled values khi clear |
| `singleSelect` | `false` | Radio thay Checkbox, chọn mới thay thế cũ |
| `changeOnBlur` | `false` | `onChange` chỉ gọi khi blur (Escape để huỷ) |
| `hideAfterSelect` | `false` | Đóng dropdown sau mỗi lần chọn |
| `hasSelectAll` | — | Option "Tất cả" |
| `maxSelectedValue` | — | Giới hạn số lượng |
| `disabledValues` | — | `unknown[]` hoặc `(opt: T) => boolean` |
| `tooltipOfDisabledOption` | — | `(opt: T) => string` |
| `tagVariant` | `"contained"` | Style tag |
| `renderOption` | — | Tuỳ chỉnh option trong dropdown |
| `renderTag` | — | Tuỳ chỉnh tag |
| `renderLimitTag` | — | Tuỳ chỉnh badge `+N` |
| `isShowIconSearch` | — | Icon search thay clear button |
| `appendToBody` | `true` | — |
| `fullWidth` | `false` | — |

### Basic usage

```tsx
<MultipleSelect
  value={selectedUsers}
  options={users}
  onChange={setSelectedUsers}
  getLabel={(user) => user.name}
  getValue={(user) => user.id}
  clearable
  fullWidth
/>
```

### Remote search

```tsx
const [searchText, setSearchText] = useState("");
<MultipleSelect
  searchValue={searchText}
  filterBySearch={false}
  onInputChange={(e) => { setSearchText(e.target.value); debouncedFetch(e.target.value); }}
  options={remoteOptions}
  optionListLoading={isLoading}
  value={selected}
  onChange={setSelected}
  getLabel={(item) => item.name}
  getValue={(item) => item.id}
/>
```

### Custom tag

```tsx
<MultipleSelect
  renderTag={({ option, onDelete, disabled }) => (
    <Tag variant="outlined" color={option.color} onDelete={!disabled ? onDelete : undefined}>
      {option.name}
    </Tag>
  )}
  ...
/>
```

### DO / DON'T

DO: `getLabel` trả về text thuần — dùng `renderOption` để tuỳ chỉnh giao diện dropdown.

DO: Dùng `disabledValues` thay vì lọc options — option vẫn hiển thị nhưng không click được.

DO: Dùng `singleSelect` khi chỉ chọn 1 — không tự giới hạn trong `onChange`.

DO: Dùng `maxSelectedValue` thay vì check `values.length` trong `onChange`.

DON'T: Truyền lại giá trị mặc định (`searchable={true}`, `disabled={false}`...) — dư thừa.

DON'T: Tự build nút clear, tag list bên ngoài — dùng `clearable`, `renderTag`.
