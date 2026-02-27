## Cách sử dụng component InsertFieldObjectButtonV2 trong ui-kit

Path: `ui-kit/src/components/InsertFieldObjectButtonV2`

Nút mở modal chọn data field. Kế thừa props từ `InsertFieldObjectV2Props`.

### Props

| Prop | Bắt buộc | Mặc định | Mô tả |
|---|---|---|---|
| `rootObjects` | ✓ | — | `InsertFieldRootObjectItem[]` |
| `onInsert` | ✓ | — | `(value: string, treeData: TreeFieldObjectV2) => void` |
| `isDisabled` | — | `false` | Disable nút mặc định |
| `children` | — | `IconButton` với `BracketsCurlyIcon` | Render prop `({ onClick }) => ReactNode` |
| `selectedDataField` | — | `null` | Tự lọc field tương thích |
| `checkSelectableField` | — | — | `(field) => boolean \| { selectable, hide }` |
| `filterDataField` | — | `() => true` | Loại field khỏi tree |
| `maxLevel` | — | `5` | Giới hạn độ sâu |
| `description` | — | i18n text | Mô tả trong modal |

### rootObjects structure

```tsx
// Mỗi item cần slug và variableSlug
rootObjects={[
  { slug: "contact", variableSlug: "$contact" },
  { slug: "currentTime", variableSlug: "$currentTime", isAdditionalOption: true },  // không query API
  { slug: "contact", variableSlug: "$contact", renderName: (obj) => obj?.name ?? "Khách hàng" },
]}
```

### Basic usage

```tsx
// Nút mặc định (IconButton với BracketsCurlyIcon)
<InsertFieldObjectButtonV2
  rootObjects={[{ slug: "contact", variableSlug: "$contact" }]}
  onInsert={(value, treeData) => {
    setSelectedVariable(value);
    setFieldType(treeData.dataField?.fieldType ?? null);
  }}
/>

// Custom button
<InsertFieldObjectButtonV2
  rootObjects={rootObjects}
  onInsert={handleInsert}
>
  {({ onClick }) => <Button variant="outlined" onClick={onClick}>Chọn trường</Button>}
</InsertFieldObjectButtonV2>

// Disable
<InsertFieldObjectButtonV2
  isDisabled={isFormDisabled}
  rootObjects={rootObjects}
  onInsert={handleInsert}
>
  {({ onClick }) => (
    <Button variant="outlined" disabled={isFormDisabled} onClick={onClick}>Chọn trường</Button>
  )}
</InsertFieldObjectButtonV2>
```

### Filter fields

```tsx
// Lọc tự động theo fieldType (recommended)
<InsertFieldObjectButtonV2 rootObjects={rootObjects} onInsert={handleInsert} selectedDataField={emailField} />

// Lọc tuỳ chỉnh
<InsertFieldObjectButtonV2
  rootObjects={rootObjects}
  onInsert={handleInsert}
  checkSelectableField={(field) => {
    if (field.fieldType === "formula") return { selectable: false, hide: true };   // ẩn
    return field.fieldType === "short_text" || field.fieldType === "long_text";
  }}
/>

// Loại bỏ khỏi tree hoàn toàn
<InsertFieldObjectButtonV2
  rootObjects={rootObjects}
  onInsert={handleInsert}
  filterDataField={(field) => field.fieldType !== "attachment"}
  maxLevel={1}  // giới hạn khi cấu trúc đơn giản
/>
```

### DO / DON'T

DO: Dùng `children` render prop để custom nút — không tự quản lý state mở/đóng modal.

DO: Dùng `isAdditionalOption: true` cho option đặc biệt (`$currentTime`, `$currentUser`) — không tạo object giả.

DO: Phân biệt `filterDataField` (lọc trước render) vs `checkSelectableField` (kiểm soát selectable sau render).

DO: Giới hạn `maxLevel` khi chỉ cần chọn field cấp 1 — tránh người dùng đi lạc vào cấu trúc sâu.

DON'T: Tự wrap div để `pointerEvents: none` — dùng `isDisabled`.

DON'T: Tự xây modal chọn field bên ngoài — dùng `children` render prop.

DON'T: Dùng `checkSelectableField` để chỉ ẩn field — dùng `filterDataField` đơn giản hơn khi chỉ cần ẩn.

DON'T: Khi cần hiển thị giá trị đã chọn + nút xoá, dùng `InsertFieldObjectButtonWrapper2` thay vì tự build.
