## Cách sử dụng component FilterGeneratorBase trong ui-kit

Path: `ui-kit/src/components/FilterGeneratorBase`

### Props

| Prop | Bắt buộc | Mặc định | Mô tả |
|---|---|---|---|
| `filterItems` | ✓ | — | Mảng filter items |
| `onChangeFilterItems` | ✓ | — | Setter cho filterItems |
| `logicType` | ✓ | — | `"AND"` \| `"OR"` \| `"CUSTOM"` \| `"NONE"` |
| `onChangeLogicType` | ✓ | — | Setter cho logicType |
| `logicValue` | ✓ | — | Biểu thức logic tuỳ chỉnh (dùng khi CUSTOM) |
| `onChangeLogicValue` | ✓ | — | Setter cho logicValue |
| `renderField` | ✓ | — | `({ value, onChange }, index) => ReactNode` |
| `renderOperators` | ✓ | — | `(field, { value, onChange }, index) => ReactNode` |
| `renderValueInput` | ✓ | — | `({ field, op }, { value, onChange }, index) => ReactNode` |
| `required` | — | `false` | Ẩn option "NONE" trong logic select |
| `disabled` | — | `false` | Disable logic select, ẩn filter list |
| `minFilterItemCount` | — | — | Disable nút xoá khi đạt giới hạn tối thiểu |
| `maxFilterItemCount` | — | `30` | Ẩn nút thêm khi đạt giới hạn |
| `showCustomLogicError` | — | `true` | Hiển thị lỗi validation cho CUSTOM logic |
| `title` | — | `"Logic kết hợp điều kiện"` | Tiêu đề trên Select logic |
| `text` | — | — | `{ fieldLabel, conditionLabel, paramLabel }` |
| `getNewParamsWhenChangeOperator` | — | — | `(op) => newParams \| undefined` |
| `getNewOperatorWhenChangeField` | — | — | `(field) => newOp \| undefined` |

### Basic usage

```tsx
interface FieldOption { label: string; value: string; }

const [filterItems, setFilterItems] = useState<FilterGeneratorBaseItem<FieldOption, string>[]>([]);
const [logicType, setLogicType] = useState<FilterGeneratorLogicTypes>("AND");
const [logicValue, setLogicValue] = useState("");

<FilterGeneratorBase<FieldOption, string>
  filterItems={filterItems}
  onChangeFilterItems={setFilterItems}
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  renderField={({ value, onChange }) => (
    <Select options={fieldOptions} value={value} onChange={onChange} fullWidth
      getLabel={(opt) => opt.label} getValue={(opt) => opt.value} />
  )}
  renderOperators={(field, { value, onChange }) => {
    const operators = field?.value === "name" ? ["=", "like"] : ["=", "!="];
    return <Select value={value} onChange={onChange} options={operators} getValue={(o) => o} getLabel={(o) => o} fullWidth />;
  }}
  renderValueInput={({ field, op }, { value, onChange }) => {
    if (!field) return null;
    return <TextField fullWidth value={(value ?? "") as string} onChange={(e) => onChange(e.target.value)} />;
  }}
/>
```

### DO / DON'T

DO: Khai báo generic type `FilterGeneratorBase<FieldOption, string>` và `FilterGeneratorBaseItem<FieldOption, string>[]` — TypeScript không infer được nếu thiếu.

DON'T: Tự build field selector bên ngoài — mất logic reset operator/params khi đổi field. Luôn dùng `renderField`.

DON'T: Tự disable nút xoá hay ẩn nút thêm bên ngoài — dùng `minFilterItemCount` và `maxFilterItemCount`.

DON'T: Tự validate CUSTOM logic bên ngoài — component tự validate, dùng `showCustomLogicError` để kiểm soát hiển thị.
