## Cách sử dụng component Select trong ui-kit

Path: `ui-kit/src/components/Select`

Kế thừa `TextFieldProps` — hỗ trợ `fullWidth`, `disabled`, `readOnly`, `placeholder`, `name`, `onFocus`, `onKeyDown`, v.v.

### Props

| Prop | Bắt buộc | Mặc định | Mô tả |
|---|---|---|---|
| `value` | ✓ | — | Option đang chọn hoặc `null` |
| `onChange` | ✓ | — | `(option \| null) => void` |
| `options` | — | — | Mảng options |
| `groupOptions` | — | — | `{ key, label, children }[]` — dùng với `mode="group"` |
| `mode` | — | `"default"` | `"default"` \| `"group"` |
| `getLabel` | — | `opt.label` | `(opt) => string` |
| `getValue` | — | `opt.value` | `(opt) => unknown` |
| `isEqual` | — | `lodash.isEqual` | So sánh tuỳ chỉnh |
| `searchable` | — | — | Bật input tìm kiếm |
| `filterBySearch` | — | `true` | `false` khi tự lọc bên ngoài |
| `onInputChange` | — | — | `(e) => void` — nhận search text |
| `clearable` | — | — | Hiển thị nút xoá |
| `disabledValues` | — | — | `unknown[]` — values bị disable |
| `renderOption` | — | — | Tuỳ chỉnh render option trong dropdown |
| `renderValue` | — | — | Tuỳ chỉnh render giá trị đã chọn |
| `height` | — | `40` | Chiều cao input (pixel) |
| `fullWidth` | — | — | Chiếm toàn bộ chiều rộng |
| `appendToBody` | — | `true` | Popper mount vào body |
| `showBorderOnBlur` | — | `true` | Hiển thị border khi blur |
| `hidePopperOnSelect` | — | `true` | Đóng dropdown sau khi chọn |
| `onScrollToBottom` | — | — | Lazy loading |
| `error` | — | — | `boolean` — border đỏ |
| `helperText` | — | — | Text phía dưới |

### Basic usage

```tsx
const [selected, setSelected] = useState<User | null>(null);

<Select
  value={selected}
  onChange={setSelected}
  options={users}
  getLabel={(opt) => opt.name}
  getValue={(opt) => opt.id}
  searchable
  clearable
  fullWidth
/>
```

### Group mode

```tsx
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

### Remote search

```tsx
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

### DO / DON'T

DO: Phân biệt `disabled` vs `readOnly` — `readOnly` trong suốt (chỉ hiển thị), `disabled` xám (vô hiệu hoá hoàn toàn).

DO: Dùng `disabledValues` thay vì lọc options — option vẫn hiển thị nhưng không chọn được.

DO: Dùng `fullWidth` thay vì `className="w-full"`.

DO: Dùng `height={32}` thay vì override CSS.

DON'T: Tự build search input, clear button, HelperText bên ngoài — dùng `searchable`, `clearable`, `error`+`helperText`.

DON'T: Tự build nhóm option bên ngoài — dùng `mode="group"` + `groupOptions`.

DON'T: Bật `searchable` và tự lọc options — bị lọc 2 lần. Dùng `filterBySearch={false}` khi tự lọc.
