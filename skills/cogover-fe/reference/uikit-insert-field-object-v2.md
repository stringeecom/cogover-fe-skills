## Cách sử dụng component InsertFieldObjectV2 trong ui-kit

Components:
- `InsertFieldObjectV2` (`ui-kit/src/components/InsertFieldObjectV2`) — modal chọn field
- `InsertFieldObjectButtonV2` (`ui-kit/src/components/InsertFieldObjectButtonV2`) — nút mở modal
- `InsertFieldObjectButtonWrapper2` (`ui-kit/src/components/InsertFieldObjectButtonWrapper2`) — wrapper tích hợp

### Props (InsertFieldObjectV2)

| Prop | Bắt buộc | Mặc định | Mô tả |
|---|---|---|---|
| `rootObjects` | ✓ | — | `InsertFieldRootObjectItem[]` |
| `onInsert` | ✓ (trừ Wrapper) | — | `(value: string, treeData: TreeFieldObjectV2) => void` |
| `selectedVariableSlug` | ✓ (Wrapper) | `null` | Slug biến đã chọn |
| `children` | ✓ (Wrapper) | — | ReactNode hiển thị khi chưa chọn |
| `onClear` | — | — | Callback nút xoá (Wrapper) |
| `onClose` | — | — | Callback đóng modal |
| `isHideButton` | — | `false` | Ẩn nút `{}` (Wrapper) |
| `isDisabled` | — | `false` | Disable nút mở modal và nút xoá |
| `selectedDataField` | — | `null` | Tự lọc field tương thích |
| `ignoreMultiple` | — | `false` | Bỏ qua kiểm tra multiple khi dùng `defaultFilterSelectableField` |
| `checkSelectableField` | — | — | `(field) => boolean \| { selectable, hide }` |
| `filterDataField` | — | `() => true` | Lọc field khỏi tree (ẩn hoàn toàn) |
| `maxLevel` | — | `5` | Giới hạn độ sâu cascader |
| `description` | — | i18n text | Mô tả trong modal |

### rootObjects structure (InsertFieldRootObjectItem)

```tsx
rootObjects={[
  { slug: "contact", variableSlug: "$contact" },              // object thường
  { slug: "currentTime", variableSlug: "$currentTime", isAdditionalOption: true },  // option đặc biệt
  { slug: "deal", variableSlug: "$deal", renderName: (obj) => obj?.name ?? "Cơ hội" },
  { slug: "contact", variableSlug: "$contact", isRootSelectable: true },  // cho phép chọn root
  { slug: "contact", variableSlug: "$relatedList", isRelatedList: true },  // hiển thị related lists
]}
```

| Field | Bắt buộc | Mô tả |
|---|---|---|
| `slug` | ✓ | Slug object trong hệ thống (dùng query API) |
| `variableSlug` | ✓ | Tên biến trong formula (`$contact`, `$deal`) |
| `isAdditionalOption` | — | `true` = không query API, hiển thị trực tiếp |
| `children` | — | Chỉ dùng khi `isAdditionalOption: true`, tự cấu hình danh sách field |
| `renderName` | — | Custom tên hiển thị: `(object: ObjectListResponse \| null) => ReactNode` |
| `isRootSelectable` | — | Cho phép chọn root item (click text = chọn, click mũi tên = mở rộng) |
| `isRelatedList` | — | Hiển thị related lists thay vì fields cho root này |

### TreeFieldObjectV2 type

```tsx
interface TreeFieldObjectV2 {
    id: string;
    name: React.ReactNode;
    slug: string;
    children?: TreeFieldObjectV2[];
    dataField?: DataField;
    isRoot?: boolean;
    isAdditionalOption?: boolean;
    isSelectable?: boolean;       // cho phép chọn item (dùng cho related list items)
    isRelatedList?: boolean;      // đánh dấu item là related list
    isRootSelectable?: boolean;   // cho phép chọn root item
}
```

### Basic usage — InsertFieldObjectButtonWrapper2 (recommended)

```tsx
const [selectedVariable, setSelectedVariable] = useState<string | null>(null);

<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={[{ slug: "contact", variableSlug: "$contact" }]}
>
  <TextField placeholder="Nhập giá trị" />
</InsertFieldObjectButtonWrapper2>
```

### Custom button — InsertFieldObjectButtonV2

```tsx
<InsertFieldObjectButtonV2
  rootObjects={rootObjects}
  onInsert={(value, treeData) => {
    setSelectedVariable(value);
    setFieldType(treeData.dataField?.fieldType ?? null);
  }}
>
  {({ onClick }) => <Button onClick={onClick}>Chọn field</Button>}
</InsertFieldObjectButtonV2>
```

### Related List

```tsx
// Hiển thị related lists thay vì fields
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={[
    { slug: "contact", variableSlug: "$relatedList", isRelatedList: true },
  ]}
>
  <TextField placeholder="Chọn related list" />
</InsertFieldObjectButtonWrapper2>
```

### Root Selectable

```tsx
// Click vào text root = chọn root, click vào mũi tên = mở rộng xem children
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={[
    { slug: "contact", variableSlug: "$contact", isRootSelectable: true },
  ]}
>
  <TextField placeholder="Chọn object hoặc field" />
</InsertFieldObjectButtonWrapper2>
```

### Filter fields

```tsx
// Lọc field tương thích tự động (cùng fieldType, cùng multiple)
<InsertFieldObjectButtonWrapper2 selectedDataField={emailField} ... />

// Lọc tương thích nhưng bỏ qua kiểm tra multiple
<InsertFieldObjectButtonWrapper2 selectedDataField={emailField} ignoreMultiple ... />

// Lọc tuỳ chỉnh — ẩn hoặc disable field
<InsertFieldObjectButtonWrapper2
  checkSelectableField={(field) => {
    if (field.fieldType === "formula") return { selectable: false, hide: true };   // ẩn
    if (field.fieldType === "long_text") return { selectable: false, hide: false }; // disable, vẫn hiện
    return true;
  }}
  ...
/>

// Loại bỏ hoàn toàn khỏi tree
<InsertFieldObjectButtonWrapper2
  filterDataField={(field) => field.fieldType !== "image" && field.fieldType !== "file"}
  ...
/>
```

### DO / DON'T

DO: Dùng `InsertFieldObjectButtonWrapper2` thay vì tự build logic mở/đóng modal + hiển thị giá trị.

DO: Phân biệt `filterDataField` (lọc trước render) vs `checkSelectableField` (sau render, có thể ẩn hoặc disable).

DO: Dùng `selectedDataField` khi chỉ cần lọc field cùng kiểu — không tự viết `checkSelectableField` cho trường hợp này.

DO: Dùng `ignoreMultiple` khi chỉ cần match fieldType mà không quan tâm multiple.

DO: Dùng `isRootSelectable` khi cần cho phép người dùng chọn cả root object, không chỉ field con.

DO: Dùng `isRelatedList` khi cần hiển thị related lists của object thay vì fields.

DON'T: Nhầm `slug` và `variableSlug` — `slug` dùng query API, `variableSlug` là tên biến (`$contact`).

DON'T: Dùng `isHideButton` ẩn nút bằng CSS `[&_button]:hidden` — dùng prop.

DON'T: Dùng `isDisabled` chỉ disable children — prop này disable cả nút mở modal và nút xoá.

DON'T: Truyền cả `selectedDataField` và `checkSelectableField` cùng lúc — `checkSelectableField` sẽ override logic mặc định.
