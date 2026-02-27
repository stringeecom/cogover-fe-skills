---
title: Cách sử dụng component InsertFieldObjectV2 trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến modal không hiển thị đúng cấu trúc dữ liệu, không lọc được field theo điều kiện, lỗi chọn field lồng nhau, và mất dữ liệu khi insert
tags: insert-field, data-field, cascader, object, variable, formula, ui-kit
---

## Cách sử dụng component InsertFieldObjectV2 trong ui-kit

Đường dẫn component: `ui-kit/src/components/InsertFieldObjectV2`

Component liên quan:
- `InsertFieldObjectButtonV2` (`ui-kit/src/components/InsertFieldObjectButtonV2`) — nút mở modal, quản lý state mở/đóng
- `InsertFieldObjectButtonWrapper2` (`ui-kit/src/components/InsertFieldObjectButtonWrapper2`) — wrapper kết hợp nút mở modal và hiển thị giá trị đã chọn

Props bắt buộc (`InsertFieldObjectV2Props`): `rootObjects`, `onInsert`.
Props tuỳ chọn: `onClose`, `checkSelectableField`, `selectedDataField`, `maxLevel`, `description`, `filterDataField`.

Props bắt buộc (`InsertFieldObjectButtonPropsV2`): `rootObjects`, `onInsert`.
Props tuỳ chọn: `isDisabled`, `children`, `checkSelectableField`, `selectedDataField`, `maxLevel`, `description`, `filterDataField`.

Props bắt buộc (`InsertFieldObjectButtonWrapperProps`): `rootObjects`, `onInsert`, `selectedVariableSlug`, `children`.
Props tuỳ chọn: `onClear`, `isHideButton`, `isDisabled`, `checkSelectableField`, `selectedDataField`, `maxLevel`, `description`, `filterDataField`.

Giá trị mặc định: `selectedDataField=null`, `description=tDataField("formula.selectDataFieldToInsert")`, `maxLevel=5`, `filterDataField=() => true`, `isDisabled=false`.

---

### RULE-INSERT-OBJECT-V2-01: Sử dụng `InsertFieldObjectButtonWrapper2` thay vì tự build UI mở modal và hiển thị giá trị đã chọn

`InsertFieldObjectButtonWrapper2` đã tích hợp sẵn logic mở modal, hiển thị giá trị đã chọn dưới dạng `TextField` disabled với nút xoá, và tooltip hiển thị slug đầy đủ. Không tự build logic mở/đóng modal và hiển thị giá trị bên ngoài.

**Sai:**

```tsx
// ❌ Tự build logic mở modal và hiển thị giá trị — thiếu logic render giá trị personnel
const [open, setOpen] = useState(false);
const [selectedVariable, setSelectedVariable] = useState<string | null>(null);

<div>
  {selectedVariable ? (
    <div className={cx("flex items-center gap-[0.5rem]")}>
      <TextField value={selectedVariable} disabled />
      <button onClick={() => setSelectedVariable(null)}>Xoá</button>
    </div>
  ) : (
    <TextField />
  )}
  <IconButton onClick={() => setOpen(true)}>
    <BracketsCurlyIcon />
  </IconButton>
  {open && (
    <InsertFieldObjectV2
      rootObjects={rootObjects}
      onClose={() => setOpen(false)}
      onInsert={(value) => {
        setSelectedVariable(value);
        setOpen(false);
      }}
    />
  )}
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng InsertFieldObjectButtonWrapper2 — đã tích hợp đầy đủ logic
const [selectedVariable, setSelectedVariable] = useState<string | null>(null);

<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={[
    { slug: "contact", variableSlug: "$contact" },
  ]}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-OBJECT-V2-02: Cấu trúc `rootObjects` phải có `slug` và `variableSlug` — không nhầm lẫn hai giá trị này

`slug` là slug của object trong hệ thống (dùng để query API lấy danh sách field). `variableSlug` là slug biến hiển thị trong formula (ví dụ: `$contact`, `$deal`). Hai giá trị này khác nhau và phải truyền đúng.

**Sai:**

```tsx
// ❌ Nhầm lẫn slug và variableSlug — slug dùng để query API, variableSlug là tên biến
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onInsert={setSelectedVariable}
  onClear={() => setSelectedVariable(null)}
  rootObjects={[
    { slug: "$contact", variableSlug: "contact" },
  ]}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

**Đúng:**

```tsx
// ✅ slug là slug object trong hệ thống, variableSlug là tên biến formula
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onInsert={setSelectedVariable}
  onClear={() => setSelectedVariable(null)}
  rootObjects={[
    { slug: "contact", variableSlug: "$contact" },
  ]}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-OBJECT-V2-03: Sử dụng `isAdditionalOption` cho các option không phải object — không tự thêm option vào danh sách

Khi cần thêm các option đặc biệt không phải object (ví dụ: `$currentTime`, `$currentUser`), sử dụng `isAdditionalOption: true` trong `rootObjects`. Component sẽ hiển thị option này mà không cần query API. Không tự render thêm option bên ngoài component.

**Sai:**

```tsx
// ❌ Tự thêm option đặc biệt bên ngoài danh sách — mất logic chọn và insert
<div>
  <button onClick={() => setSelectedVariable("$currentTime")}>Current Time</button>
  <InsertFieldObjectButtonWrapper2
    selectedVariableSlug={selectedVariable}
    onInsert={setSelectedVariable}
    onClear={() => setSelectedVariable(null)}
    rootObjects={[
      { slug: "contact", variableSlug: "$contact" },
    ]}
  >
    <TextField />
  </InsertFieldObjectButtonWrapper2>
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng isAdditionalOption cho option đặc biệt
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onInsert={setSelectedVariable}
  onClear={() => setSelectedVariable(null)}
  rootObjects={[
    { slug: "contact", variableSlug: "$contact" },
    { slug: "currentTime", variableSlug: "$currentTime", isAdditionalOption: true },
  ]}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-OBJECT-V2-04: Sử dụng `renderName` trong `rootObjects` để tuỳ chỉnh tên hiển thị — không tự map tên bên ngoài

`renderName` nhận vào `ObjectListResponse | null` (null khi `isAdditionalOption=true`) và trả về `React.ReactNode`. Sử dụng prop này để tuỳ chỉnh tên hiển thị của root object. Không tự map tên bên ngoài.

**Sai:**

```tsx
// ❌ Tự đổi tên object bên ngoài — mất đồng bộ với giao diện cascader
const customNames: Record<string, string> = { contact: "Khách hàng", deal: "Cơ hội" };

<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onInsert={setSelectedVariable}
  onClear={() => setSelectedVariable(null)}
  rootObjects={objects.map((obj) => ({
    slug: obj.slug,
    variableSlug: `$${customNames[obj.slug] ?? obj.slug}`,
  }))}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

**Đúng:**

```tsx
// ✅ Sử dụng renderName để tuỳ chỉnh tên hiển thị
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onInsert={setSelectedVariable}
  onClear={() => setSelectedVariable(null)}
  rootObjects={[
    {
      slug: "contact",
      variableSlug: "$contact",
      renderName: (object) => object?.name ?? "Khách hàng",
    },
    {
      slug: "deal",
      variableSlug: "$deal",
      renderName: (object) => object?.name ?? "Cơ hội",
    },
  ]}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-OBJECT-V2-05: Sử dụng `selectedDataField` để tự động lọc field tương thích — không tự lọc bằng `filterDataField`

Khi truyền `selectedDataField`, component sử dụng hàm `defaultFilterSelectableField` để tự động lọc các field tương thích (cùng `fieldType`, cùng `multiple`, kiểm tra formula return type). Chỉ dùng `checkSelectableField` khi cần logic lọc hoàn toàn khác.

**Sai:**

```tsx
// ❌ Tự lọc field bằng filterDataField để kiểm tra tương thích — thiếu logic formula, lookup
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onInsert={setSelectedVariable}
  onClear={() => setSelectedVariable(null)}
  filterDataField={(field) => field.fieldType === emailField?.fieldType}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

**Đúng:**

```tsx
// ✅ Truyền selectedDataField để component tự lọc field tương thích
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onInsert={setSelectedVariable}
  onClear={() => setSelectedVariable(null)}
  selectedDataField={emailField}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-OBJECT-V2-06: Sử dụng `checkSelectableField` khi cần logic lọc tuỳ chỉnh — trả về đúng kiểu dữ liệu

`checkSelectableField` nhận vào `DataField` và trả về `boolean` hoặc `{ selectable: boolean; hide: boolean }`. Khi trả về object, `hide=true` sẽ ẩn field khỏi danh sách, `selectable=false` sẽ hiển thị field nhưng không cho chọn (vẫn cho phép mở rộng nếu là lookup field). Không nhầm lẫn `checkSelectableField` với `filterDataField`.

**Sai:**

```tsx
// ❌ Dùng filterDataField để disable field — filterDataField chỉ lọc hiển thị, không disable
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onInsert={setSelectedVariable}
  onClear={() => setSelectedVariable(null)}
  filterDataField={(field) => field.fieldType !== "formula"}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

**Đúng:**

```tsx
// ✅ Dùng checkSelectableField với object trả về để ẩn hoặc disable field
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onInsert={setSelectedVariable}
  onClear={() => setSelectedVariable(null)}
  checkSelectableField={(field) => {
    if (field.fieldType === "formula") {
      return { selectable: false, hide: true };
    }
    if (field.fieldType === "long_text") {
      return { selectable: false, hide: false };
    }
    return true;
  }}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-OBJECT-V2-07: Sử dụng `filterDataField` để lọc field hiển thị — không nhầm với `checkSelectableField`

`filterDataField` nhận vào `DataField` và trả về `boolean`. Khi trả về `false`, field sẽ bị ẩn hoàn toàn khỏi cây cascader. Dùng khi muốn loại bỏ một số loại field khỏi danh sách (ví dụ: chỉ hiển thị field text). Khác với `checkSelectableField` — `filterDataField` lọc trước khi build tree, còn `checkSelectableField` lọc sau khi render.

**Sai:**

```tsx
// ❌ Dùng checkSelectableField chỉ để ẩn field — nên dùng filterDataField cho đơn giản
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onInsert={setSelectedVariable}
  onClear={() => setSelectedVariable(null)}
  checkSelectableField={(field) => {
    if (field.fieldType === "image" || field.fieldType === "file") {
      return { selectable: false, hide: true };
    }
    return true;
  }}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

**Đúng:**

```tsx
// ✅ Dùng filterDataField để loại bỏ field không cần hiển thị
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onInsert={setSelectedVariable}
  onClear={() => setSelectedVariable(null)}
  filterDataField={(field) => field.fieldType !== "image" && field.fieldType !== "file"}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-OBJECT-V2-08: Sử dụng `maxLevel` để giới hạn độ sâu cascader — không tự giới hạn bằng logic bên ngoài

Mặc định `maxLevel=5`. Khi cần giới hạn độ sâu của cây cascader (ví dụ: không cho phép chọn field lồng quá 2 cấp), truyền `maxLevel`. Component tự xử lý logic ẩn mũi tên mở rộng khi đạt giới hạn. Không tự viết logic giới hạn bên ngoài.

**Sai:**

```tsx
// ❌ Tự giới hạn bằng checkSelectableField — phức tạp và không chính xác
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onInsert={setSelectedVariable}
  onClear={() => setSelectedVariable(null)}
  checkSelectableField={(field) => {
    // Tự đếm depth để giới hạn — không đúng cách
    const depth = selectedVariable?.split(".").length ?? 0;
    if (depth >= 2) return { selectable: false, hide: true };
    return true;
  }}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

**Đúng:**

```tsx
// ✅ Sử dụng maxLevel để giới hạn độ sâu cascader
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onInsert={setSelectedVariable}
  onClear={() => setSelectedVariable(null)}
  maxLevel={2}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-OBJECT-V2-09: Callback `onInsert` nhận `(value: string, treeData: TreeFieldObjectV2)` — sử dụng đúng tham số

`onInsert` nhận hai tham số: `value` là variable slug đầy đủ (ví dụ: `$contact.email`), `treeData` là object chứa thông tin chi tiết của field đã chọn (bao gồm `dataField`, `name`, `slug`). Sử dụng `treeData` khi cần thêm thông tin ngoài slug.

**Sai:**

```tsx
// ❌ Chỉ dùng value và bỏ qua treeData — mất thông tin field type
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onInsert={(value) => {
    setSelectedVariable(value);
    // Không biết field type của giá trị đã chọn
  }}
  onClear={() => setSelectedVariable(null)}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

**Đúng:**

```tsx
// ✅ Sử dụng cả value và treeData khi cần thông tin chi tiết
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onInsert={(value, treeData) => {
    setSelectedVariable(value);
    setSelectedFieldType(treeData.dataField?.fieldType ?? null);
  }}
  onClear={() => setSelectedVariable(null)}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-OBJECT-V2-10: Sử dụng `InsertFieldObjectButtonV2` khi cần tuỳ chỉnh nút mở modal — không tự quản lý state mở/đóng

`InsertFieldObjectButtonV2` quản lý state mở/đóng modal. Prop `children` là render function nhận `{ onClick }` để tuỳ chỉnh nút bấm. Không tự quản lý state `openInsertModal` bên ngoài.

**Sai:**

```tsx
// ❌ Tự quản lý state mở/đóng modal
const [openModal, setOpenModal] = useState(false);

<Button onClick={() => setOpenModal(true)}>Chọn field</Button>
{openModal && (
  <InsertFieldObjectV2
    rootObjects={rootObjects}
    onClose={() => setOpenModal(false)}
    onInsert={(value, treeData) => {
      setSelectedVariable(value);
      setOpenModal(false);
    }}
  />
)}
```

**Đúng:**

```tsx
// ✅ Sử dụng InsertFieldObjectButtonV2 với custom children
<InsertFieldObjectButtonV2
  rootObjects={rootObjects}
  onInsert={(value) => setSelectedVariable(value)}
>
  {({ onClick }) => (
    <Button onClick={onClick}>Chọn field</Button>
  )}
</InsertFieldObjectButtonV2>
```

---

### RULE-INSERT-OBJECT-V2-11: Sử dụng `isHideButton` để ẩn nút mở modal trong Wrapper — không tự ẩn bằng CSS

Khi cần ẩn nút mở modal (ví dụ: chỉ đọc, không cho phép thay đổi), sử dụng `isHideButton={true}` trên `InsertFieldObjectButtonWrapper2`. Không tự ẩn nút bằng CSS.

**Sai:**

```tsx
// ❌ Ẩn nút bằng CSS — nút vẫn render trong DOM
<InsertFieldObjectButtonWrapper2
  className={cx("[&_button]:hidden")}
  selectedVariableSlug={selectedVariable}
  onInsert={setSelectedVariable}
  onClear={() => setSelectedVariable(null)}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

**Đúng:**

```tsx
// ✅ Sử dụng isHideButton để ẩn nút
<InsertFieldObjectButtonWrapper2
  isHideButton
  selectedVariableSlug={selectedVariable}
  onInsert={setSelectedVariable}
  onClear={() => setSelectedVariable(null)}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-OBJECT-V2-12: Sử dụng `isDisabled` để vô hiệu hoá nút mở modal — không tự disable children

Khi truyền `isDisabled={true}`, nút mở modal (`IconButton`) sẽ bị disable và nút xoá giá trị đã chọn cũng bị disable. Không tự disable component children bên ngoài.

**Sai:**

```tsx
// ❌ Tự disable children bên ngoài — nút mở modal và nút xoá vẫn hoạt động
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onInsert={setSelectedVariable}
  onClear={() => setSelectedVariable(null)}
  rootObjects={rootObjects}
>
  <TextField disabled />
</InsertFieldObjectButtonWrapper2>
```

**Đúng:**

```tsx
// ✅ Sử dụng isDisabled — disable cả nút mở modal và nút xoá
<InsertFieldObjectButtonWrapper2
  isDisabled
  selectedVariableSlug={selectedVariable}
  onInsert={setSelectedVariable}
  onClear={() => setSelectedVariable(null)}
  rootObjects={rootObjects}
>
  <TextField disabled />
</InsertFieldObjectButtonWrapper2>
```

---

| Prop | Giá trị mặc định | Mô tả |
|---|---|---|
| `selectedDataField` | `null` | DataField đang được gán — dùng để lọc field tương thích tự động |
| `description` | `tDataField("formula.selectDataFieldToInsert")` | Mô tả hiển thị phía trên cây cascader |
| `maxLevel` | `5` | Giới hạn độ sâu tối đa của cây cascader |
| `filterDataField` | `() => true` | Hàm lọc field hiển thị trong cây cascader |
| `checkSelectableField` | `defaultFilterSelectableField` (dựa trên `selectedDataField`) | Hàm kiểm tra field có thể chọn được không |
| `isDisabled` | `false` | Vô hiệu hoá nút mở modal và nút xoá |
| `isHideButton` | `undefined` | Ẩn nút mở modal trong Wrapper |
