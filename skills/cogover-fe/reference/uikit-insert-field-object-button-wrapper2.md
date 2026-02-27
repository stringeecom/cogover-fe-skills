## Cách sử dụng component InsertFieldObjectButtonWrapper2 trong ui-kit

Path: `ui-kit/src/components/InsertFieldObjectButtonWrapper2`

Wrapper tích hợp: hiển thị `children` khi chưa chọn biến, hiển thị `TextField` disabled với giá trị + nút xoá khi đã chọn.

### Props

| Prop | Bắt buộc | Mặc định | Mô tả |
|---|---|---|---|
| `children` | ✓ | — | ReactNode — hiển thị khi `selectedVariableSlug` rỗng |
| `selectedVariableSlug` | ✓ | `null` | Slug biến đã chọn |
| `rootObjects` | ✓ | — | `InsertFieldRootObjectItem[]` |
| `onClear` | — | — | Callback nút xoá (hiện khi có giá trị) |
| `onInsert` | — | — | `(value: string, treeData) => void` |
| `isHideButton` | — | `false` | Ẩn nút `{}` |
| `isDisabled` | — | `false` | Disable cả nút `{}` và nút xoá |
| `selectedDataField` | — | `null` | Tự lọc field tương thích |
| `checkSelectableField` | — | — | `(field) => boolean \| { selectable, hide }` |
| `filterDataField` | — | — | Lọc field khỏi tree |
| `maxLevel` | — | — | Giới hạn độ sâu |
| `description` | — | — | Mô tả trong modal |

### rootObjects structure

```tsx
rootObjects={[
  { slug: "contact", variableSlug: "$contact" },
  { slug: "currentTime", variableSlug: "$currentTime", isAdditionalOption: true },
]}
```

### Basic usage

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

### Với filter field

```tsx
// Lọc tự động (recommended)
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  selectedDataField={emailField}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>

// Lọc tuỳ chỉnh phức tạp (chỉ khi selectedDataField không đủ)
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={rootObjects}
  checkSelectableField={(field) => {
    if (field.fieldType === "lookup") return { selectable: false, hide: false };
    return field.fieldType === "text" || field.fieldType === "email";
  }}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

### Ẩn nút + disable

```tsx
// Ẩn nút {} (chỉ xem, không cho chọn thêm)
<InsertFieldObjectButtonWrapper2 isHideButton selectedVariableSlug={selectedVariable} onClear={...} rootObjects={rootObjects}>
  <TextField />
</InsertFieldObjectButtonWrapper2>

// Disable cả nút {} và nút xoá
<InsertFieldObjectButtonWrapper2 isDisabled={isFormDisabled} selectedVariableSlug={selectedVariable} onClear={...} onInsert={...} rootObjects={rootObjects}>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

### Giới hạn kích thước

```tsx
// Wrapper chiếm w-full — bọc trong container để kiểm soát chiều rộng
<div className={cx("w-[12.5rem]")}>
  <InsertFieldObjectButtonWrapper2 selectedVariableSlug={selectedVariable} onClear={...} onInsert={...} rootObjects={rootObjects}>
    <TextField />
  </InsertFieldObjectButtonWrapper2>
</div>
```

### DO / DON'T

DO: Luôn truyền `children` — component rỗng khi chưa chọn nếu không có children.

DO: Truyền `onClear` cùng với `selectedVariableSlug` — component hiển thị nút xoá dựa trên `onClear`.

DO: Dùng `selectedDataField` khi chỉ cần lọc field cùng kiểu — không tự viết `checkSelectableField`.

DON'T: Không truyền cả `selectedDataField` và `checkSelectableField` cùng lúc — logic conflict.

DON'T: Ẩn nút bằng CSS `[&_button]:hidden` — dùng `isHideButton`.

DON'T: Override width trực tiếp lên wrapper — bọc trong container.

DON'T: Tự build nút clear bên ngoài — component tự hiển thị nút xoá khi `selectedVariableSlug` có giá trị.
