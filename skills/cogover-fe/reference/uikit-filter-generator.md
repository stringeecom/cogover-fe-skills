## Cách sử dụng component FilterGenerator trong ui-kit

Path: `ui-kit/src/components/FilterGenerator`

### Props

| Prop | Bắt buộc | Mặc định | Mô tả |
|---|---|---|---|
| `dataFields` | ✓ | — | `DataField[]` từ API |
| `logicType` | ✓ | — | `"AND"` \| `"OR"` \| `"CUSTOM"` \| `"NONE"` |
| `logicValue` | ✓ | — | Biểu thức logic (dùng khi CUSTOM) |
| `filterItems` | ✓ | — | `FilterGeneratorItem[]` |
| `onChangeLogicType` | ✓ | — | Setter |
| `onChangeLogicValue` | ✓ | — | Setter |
| `onChangeFilterItems` | ✓ | — | Setter |
| `recordDataFields` | — | — | DataField[] cho record — component tự nhóm với prefix `$record.` |
| `required` | — | `false` | Ẩn option "NONE" |
| `disabled` | — | `false` | Ẩn toàn bộ filter list, disable logic select |
| `readOnly` | — | `false` | Hiển thị nhưng không cho chỉnh sửa |
| `clearable` | — | — | Hiển thị nút Reset |
| `onReset` | — | — | Callback khi reset |
| `maxFilterItemCount` | — | `30` | Giới hạn số điều kiện |
| `showCustomLogicError` | — | `true` | Validate & hiển thị lỗi CUSTOM logic |
| `showPreview` | — | `false` | Hiển thị nút xem trước |
| `onClickPreview` | — | — | Handler nút xem trước |
| `disabledPreviewButton` | — | `false` | Disable nút xem trước |
| `insertFieldObjectButtonWrapperProps` | — | — | Props cho variable picker |
| `renderValueInputWrapper` | — | — | `({ filterItem, onChangeFilterItem, children }) => ReactNode` |
| `isWorkflow` | — | `false` | Bật chế độ workflow |
| `text` | — | — | `{ objectDataFieldLabel, recordDataFieldLabel }` |

### Basic usage

```tsx
const [logicType, setLogicType] = useState<FilterGeneratorLogicTypes>("AND");
const [logicValue, setLogicValue] = useState("");
const [filterItems, setFilterItems] = useState<FilterGeneratorItem[]>([]);
const { data: object } = useQueryObjectDetailBySlug({ slug: objectSlug });

<FilterGenerator
  dataFields={object?.fields ?? []}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
  clearable
  onReset={() => { /* gọi API nếu cần */ }}
/>
```

### FilterGeneratorItem structure

```tsx
const [filterItems, setFilterItems] = useState<FilterGeneratorItem[]>([
  {
    field: "created_by",    // slug của trường hoặc null
    op: "=",               // Operator hoặc null
    params: "$currentRecord",
    fieldType: "lookup_normal",
    iu: false,             // is current user value (optional)
  },
]);
```

### DO / DON'T

DO: Phân biệt `disabled` vs `readOnly` — `disabled` ẩn toàn bộ list, `readOnly` hiển thị nhưng không cho sửa.

DO: Truyền `recordDataFields` riêng thay vì gộp vào `dataFields` — component tự nhóm và thêm prefix.

DON'T: Tự build logic select (AND/OR/CUSTOM), nút reset, nút preview, variable picker — dùng props có sẵn.

DON'T: Tự validate CUSTOM logic expression — dùng `showCustomLogicError`.

DON'T: Tự giới hạn số điều kiện trong `onChangeFilterItems` — dùng `maxFilterItemCount`.

DON'T: Tự theo dõi thay đổi params bằng `useEffect` so sánh mảng — dùng `onChangeFilterItemParams`.
