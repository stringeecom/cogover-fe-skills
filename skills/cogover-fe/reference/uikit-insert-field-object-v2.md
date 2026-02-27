## Cách sử dụng component InsertFieldObjectV2 trong ui-kit

Components:
- `InsertFieldObjectV2` (`ui-kit/src/components/InsertFieldObjectV2`) — modal chọn field
- `InsertFieldObjectButtonV2` (`ui-kit/src/components/InsertFieldObjectButtonV2`) — nút mở modal
- `InsertFieldObjectButtonWrapper2` (`ui-kit/src/components/InsertFieldObjectButtonWrapper2`) — wrapper tích hợp

### Props

| Prop | Bắt buộc | Mặc định | Mô tả |
|---|---|---|---|
| `rootObjects` | ✓ | — | `InsertFieldRootObjectItem[]` |
| `onInsert` | ✓ (trừ Wrapper) | — | `(value: string, treeData: TreeFieldObjectV2) => void` |
| `selectedVariableSlug` | ✓ (Wrapper) | `null` | Slug biến đã chọn |
| `children` | ✓ (Wrapper) | — | ReactNode hiển thị khi chưa chọn |
| `onClear` | — | — | Callback nút xoá (Wrapper) |
| `isHideButton` | — | `false` | Ẩn nút `{}` (Wrapper) |
| `isDisabled` | — | `false` | Disable nút mở modal và nút xoá |
| `selectedDataField` | — | `null` | Tự lọc field tương thích |
| `checkSelectableField` | — | — | `(field) => boolean \| { selectable, hide }` |
| `filterDataField` | — | `() => true` | Lọc field khỏi tree (ẩn hoàn toàn) |
| `maxLevel` | — | `5` | Giới hạn độ sâu cascader |
| `description` | — | i18n text | Mô tả trong modal |

### rootObjects structure

```tsx
rootObjects={[
  { slug: "contact", variableSlug: "$contact" },              // object thường
  { slug: "currentTime", variableSlug: "$currentTime", isAdditionalOption: true },  // option đặc biệt
  { slug: "deal", variableSlug: "$deal", renderName: (obj) => obj?.name ?? "Cơ hội" },
]}
```

- `slug`: slug object trong hệ thống (dùng query API)
- `variableSlug`: tên biến trong formula (`$contact`, `$deal`)
- `isAdditionalOption: true`: không query API, hiển thị trực tiếp

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

### Filter fields

```tsx
// Lọc field tương thích tự động (cùng fieldType, cùng multiple)
<InsertFieldObjectButtonWrapper2 selectedDataField={emailField} ... />

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

DON'T: Nhầm `slug` và `variableSlug` — `slug` dùng query API, `variableSlug` là tên biến (`$contact`).

DON'T: Dùng `isHideButton` ẩn nút bằng CSS `[&_button]:hidden` — dùng prop.

DON'T: Dùng `isDisabled` chỉ disable children — prop này disable cả nút mở modal và nút xoá.
