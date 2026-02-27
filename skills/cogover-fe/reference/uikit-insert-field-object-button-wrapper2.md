---
title: Cách sử dụng component InsertFieldObjectButtonWrapper2 trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến không hiển thị được biến đã chọn, mất chức năng clear, lỗi khi truyền rootObjects, và giao diện wrapper không đồng bộ với InsertFieldObjectButtonV2
tags: insert-field, variable, field-object, wrapper, ui-kit
---

## Cách sử dụng component InsertFieldObjectButtonWrapper2 trong ui-kit

Đường dẫn component: `ui-kit/src/components/InsertFieldObjectButtonWrapper2`

Props bắt buộc: `children`, `selectedVariableSlug`, `rootObjects`.
Props tuỳ chọn: `onClear`, `isHideButton`, `onInsert`, `isDisabled`, `checkSelectableField`, `selectedDataField`, `maxLevel`, `description`, `filterDataField`.

Kế thừa từ `InsertFieldObjectButtonPropsV2` (ngoại trừ `children`) nên hỗ trợ tất cả props của `InsertFieldObjectV2Props` (ngoại trừ `object`, `onClose`).

Giá trị mặc định: `selectedVariableSlug=null`, `isHideButton=undefined`, `isDisabled=false`.

---

### RULE-INSERT-WRAPPER2-01: Phải truyền `children` làm nội dung mặc định khi chưa chọn biến

`InsertFieldObjectButtonWrapper2` hiển thị `children` khi `selectedVariableSlug` là `null` hoặc chuỗi rỗng. Khi `selectedVariableSlug` có giá trị, component tự động hiển thị `TextField` disabled với giá trị biến đã chọn. Phải truyền `children` để hiển thị giao diện mặc định.

**Sai:**

```tsx
// ❌ Không truyền children — khi chưa chọn biến, wrapper rỗng không hiển thị gì
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={rootObjects}
/>
```

**Đúng:**

```tsx
// ✅ Truyền children làm nội dung mặc định
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={rootObjects}
>
  <TextField placeholder="Nhập giá trị" />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-WRAPPER2-02: `selectedVariableSlug` và `onClear` phải đi cùng nhau — không quản lý clear bên ngoài

Khi `selectedVariableSlug` có giá trị, component tự động hiển thị nút xoá (IconButton với icon faXmark) bên trong TextField. Nút xoá sẽ gọi `onClear`. Không tự build nút clear bên ngoài wrapper.

**Sai:**

```tsx
// ❌ Tự build nút clear bên ngoài — trùng lặp với nút clear có sẵn
<div className={cx("flex items-center gap-[0.5rem]")}>
  <InsertFieldObjectButtonWrapper2
    selectedVariableSlug={selectedVariable}
    onInsert={setSelectedVariable}
    rootObjects={rootObjects}
  >
    <TextField />
  </InsertFieldObjectButtonWrapper2>
  <button onClick={() => setSelectedVariable(null)}>Xoá</button>
</div>
```

**Đúng:**

```tsx
// ✅ Truyền onClear — component tự hiển thị nút xoá khi có biến được chọn
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-WRAPPER2-03: `rootObjects` phải đúng định dạng `InsertFieldRootObjectItem[]` — mỗi item cần `slug` và `variableSlug`

`rootObjects` là mảng các object gốc để hiển thị danh sách chọn biến. Mỗi item cần `slug` (slug của object trong hệ thống) và `variableSlug` (tiền tố biến, ví dụ `$contact`). Truyền `isAdditionalOption: true` cho các option không phải object thực (ví dụ: `$currentTime`).

**Sai:**

```tsx
// ❌ Truyền sai định dạng rootObjects — thiếu variableSlug
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={[
    { slug: "contact" },
    { slug: "currentTime" },
  ]}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

**Đúng:**

```tsx
// ✅ Truyền đúng định dạng với slug và variableSlug
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={[
    {
      slug: "contact",
      variableSlug: "$contact",
    },
    {
      slug: "currentTime",
      variableSlug: "$currentTime",
      isAdditionalOption: true,
    },
  ]}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-WRAPPER2-04: Sử dụng `onInsert` để nhận giá trị biến đã chọn — không tự build modal chọn biến

`onInsert` được kế thừa từ `InsertFieldObjectButtonV2` và được gọi khi người dùng chọn một biến từ modal. Callback nhận `value` (chuỗi slug của biến) và `treeData` (dữ liệu tree node). Không tự build modal chọn biến bên ngoài.

**Sai:**

```tsx
// ❌ Tự build modal chọn biến bên ngoài — mất logic tree, search, filter
const [showModal, setShowModal] = useState(false);

<div>
  <InsertFieldObjectButtonWrapper2
    isHideButton
    selectedVariableSlug={selectedVariable}
    onClear={() => setSelectedVariable(null)}
    rootObjects={rootObjects}
  >
    <TextField />
  </InsertFieldObjectButtonWrapper2>
  <button onClick={() => setShowModal(true)}>Chọn biến</button>
  {showModal && <CustomFieldModal onSelect={setSelectedVariable} />}
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng onInsert — component tự quản lý modal chọn biến
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-WRAPPER2-05: Sử dụng `isHideButton` để ẩn nút chèn biến — không tự ẩn bằng CSS

Khi không muốn hiển thị nút chèn biến (icon `{}`), truyền `isHideButton={true}`. Component sẽ không render `InsertFieldObjectButtonV2`. Không override CSS để ẩn nút.

**Sai:**

```tsx
// ❌ Ẩn nút bằng CSS — nút vẫn render, chiếm DOM, có thể gây side effect
<InsertFieldObjectButtonWrapper2
  className={cx("[&_button]:hidden")}
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

**Đúng:**

```tsx
// ✅ Sử dụng isHideButton — nút không render
<InsertFieldObjectButtonWrapper2
  isHideButton
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-WRAPPER2-06: Sử dụng `isDisabled` để vô hiệu hoá nút chèn biến và nút clear — không tự quản lý trạng thái disabled

Khi `isDisabled=true`, nút chèn biến (`InsertFieldObjectButtonV2`) bị vô hiệu hoá và nút clear (IconButton) bên trong TextField disabled cũng bị vô hiệu hoá. Không tự quản lý trạng thái disabled bên ngoài.

**Sai:**

```tsx
// ❌ Tự quản lý disabled — chỉ disable được nút ngoài, nút clear bên trong vẫn hoạt động
<div className={cx({ "pointer-events-none opacity-50": isFormDisabled })}>
  <InsertFieldObjectButtonWrapper2
    selectedVariableSlug={selectedVariable}
    onClear={() => setSelectedVariable(null)}
    onInsert={setSelectedVariable}
    rootObjects={rootObjects}
  >
    <TextField />
  </InsertFieldObjectButtonWrapper2>
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng isDisabled — cả nút chèn biến và nút clear đều bị vô hiệu hoá
<InsertFieldObjectButtonWrapper2
  isDisabled={isFormDisabled}
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-WRAPPER2-07: Sử dụng `selectedDataField` để lọc field mặc định — không tự build logic filter

Khi truyền `selectedDataField`, component sử dụng hàm filter mặc định bên trong `InsertFieldObjectV2` để chỉ hiển thị các field tương thích với `dataField` đã truyền. Không tự build logic filter bên ngoài khi chỉ cần lọc theo kiểu field.

**Sai:**

```tsx
// ❌ Tự build logic filter — trùng lặp với logic có sẵn
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={rootObjects}
  checkSelectableField={(field) => field.fieldType === "text"}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

**Đúng:**

```tsx
// ✅ Truyền selectedDataField — component tự lọc field tương thích
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  selectedDataField={emailField}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-WRAPPER2-08: Sử dụng `checkSelectableField` khi cần logic lọc tuỳ chỉnh — không dùng khi `selectedDataField` đủ

`checkSelectableField` cho phép tuỳ chỉnh logic lọc field phức tạp. Hàm nhận `DataField` và trả về `boolean` hoặc `{ selectable: boolean; hide: boolean }`. Chỉ dùng khi `selectedDataField` không đáp ứng được yêu cầu lọc. Khi trả về object, `hide: true` sẽ ẩn field khỏi danh sách, `selectable: false` sẽ hiển thị nhưng không cho chọn.

**Sai:**

```tsx
// ❌ Truyền cả selectedDataField và checkSelectableField — logic bị conflict
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  selectedDataField={emailField}
  checkSelectableField={(field) => field.fieldType === "text"}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

**Đúng:**

```tsx
// ✅ Chỉ dùng checkSelectableField khi cần logic tuỳ chỉnh phức tạp
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={rootObjects}
  checkSelectableField={(field) => {
    if (field.fieldType === "lookup") {
      return { selectable: false, hide: false };
    }
    return field.fieldType === "text" || field.fieldType === "email";
  }}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-WRAPPER2-09: Sử dụng `maxLevel` để giới hạn độ sâu tree — không tự cắt dữ liệu rootObjects

`maxLevel` giới hạn số cấp hiển thị trong tree chọn biến bên trong `InsertFieldObjectV2`. Không tự cắt bớt children trong `rootObjects` để giới hạn độ sâu.

**Sai:**

```tsx
// ❌ Tự cắt bớt children — mất dữ liệu, logic không nhất quán
const trimmedRootObjects = rootObjects.map((obj) => ({
  ...obj,
  children: obj.children?.map((child) => ({ ...child, children: undefined })),
}));

<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={trimmedRootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

**Đúng:**

```tsx
// ✅ Sử dụng maxLevel để giới hạn độ sâu
<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={rootObjects}
  maxLevel={2}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

### RULE-INSERT-WRAPPER2-10: Wrapper chiếm toàn bộ chiều rộng parent — bọc trong container để kiểm soát kích thước

Component sử dụng `className={cx("w-full", "inline-flex")}` nên tự động chiếm toàn bộ chiều rộng parent. Children được bọc trong `flex-1 w-0` để co giãn. Khi cần giới hạn chiều rộng, bọc wrapper trong container có width cụ thể.

**Sai:**

```tsx
// ❌ Override width trực tiếp lên wrapper — dễ conflict với layout nội bộ
<InsertFieldObjectButtonWrapper2
  className={cx("w-[12.5rem]")}
  selectedVariableSlug={selectedVariable}
  onClear={() => setSelectedVariable(null)}
  onInsert={setSelectedVariable}
  rootObjects={rootObjects}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

**Đúng:**

```tsx
// ✅ Bọc trong container để kiểm soát kích thước
<div className={cx("w-[12.5rem]")}>
  <InsertFieldObjectButtonWrapper2
    selectedVariableSlug={selectedVariable}
    onClear={() => setSelectedVariable(null)}
    onInsert={setSelectedVariable}
    rootObjects={rootObjects}
  >
    <TextField />
  </InsertFieldObjectButtonWrapper2>
</div>
```

---

### Bảng tham chiếu giá trị mặc định

| Prop | Giá trị mặc định | Mô tả |
|---|---|---|
| `selectedVariableSlug` | `null` | Slug biến đã chọn, `null` khi chưa chọn |
| `isHideButton` | `undefined` (falsy) | Ẩn nút chèn biến `{}` |
| `isDisabled` | `false` | Vô hiệu hoá nút chèn biến và nút clear |
| `maxLevel` | `undefined` | Giới hạn độ sâu tree chọn biến |
| `selectedDataField` | `null` | DataField dùng để lọc field mặc định |
| `checkSelectableField` | `undefined` | Hàm tuỳ chỉnh logic lọc field |
| `filterDataField` | `undefined` | Hàm lọc DataField hiển thị trong tree |
| `description` | `undefined` | Nội dung mô tả hiển thị trong modal chọn biến |
