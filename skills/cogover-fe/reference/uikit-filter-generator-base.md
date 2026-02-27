---
title: Cách sử dụng component FilterGeneratorBase trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến không render được bộ lọc, mất logic kết hợp điều kiện, lỗi TypeScript generic, và hành vi thêm/xoá điều kiện không nhất quán
tags: filter, filter-generator, condition, logic, AND, OR, CUSTOM, ui-kit
---

## Cách sử dụng component FilterGeneratorBase trong ui-kit

Đường dẫn component: `ui-kit/src/components/FilterGeneratorBase`

Props bắt buộc: `filterItems`, `onChangeFilterItems`, `logicType`, `onChangeLogicType`, `logicValue`, `onChangeLogicValue`, `renderField`, `renderOperators`, `renderValueInput`.
Props tuỳ chọn: `required`, `disabled`, `minFilterItemCount`, `maxFilterItemCount`, `showCustomLogicError`, `title`, `text`, `getNewParamsWhenChangeOperator`, `getNewOperatorWhenChangeField`.

Kế thừa từ `React.HTMLAttributes<HTMLDivElement>` nên hỗ trợ: `className`, `style`, `id`, `data-testid`, và các thuộc tính HTML div khác.

Giá trị mặc định: `required=false`, `disabled=false`, `showCustomLogicError=true`, `maxFilterItemCount=30`, `title="Logic kết hợp điều kiện"`.

---

### RULE-FILTER-GENERATOR-BASE-01: FilterGeneratorBase là controlled component — phải truyền đầy đủ state và callback

`FilterGeneratorBase` là controlled component với 3 cặp state/callback: `filterItems`/`onChangeFilterItems`, `logicType`/`onChangeLogicType`, `logicValue`/`onChangeLogicValue`. Phải quản lý state bên ngoài và truyền đầy đủ.

**Sai:**

```tsx
// ❌ Thiếu state và callback — component không hoạt động
<FilterGeneratorBase
  filterItems={[]}
  renderField={() => null}
  renderOperators={() => null}
  renderValueInput={() => null}
/>
```

**Đúng:**

```tsx
// ✅ Controlled component với đầy đủ state và callback
const [filterItems, setFilterItems] = useState<FilterGeneratorBaseItem<Option, string>[]>([]);
const [logicType, setLogicType] = useState<FilterGeneratorLogicTypes>("AND");
const [logicValue, setLogicValue] = useState("");

<FilterGeneratorBase
  filterItems={filterItems}
  onChangeFilterItems={setFilterItems}
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  renderField={({ value, onChange }) => (
    <Select
      options={fieldOptions}
      value={value}
      onChange={onChange}
      fullWidth
      getLabel={(opt) => opt.label}
      getValue={(opt) => opt.value}
    />
  )}
  renderOperators={(field, { value, onChange }) => (
    <Select
      value={value}
      onChange={onChange}
      options={getOperatorsByField(field)}
      getValue={(o) => o}
      getLabel={(o) => o}
      fullWidth
    />
  )}
  renderValueInput={({ field, op }, { value, onChange }) => {
    if (!field) return null;
    return (
      <TextField
        fullWidth
        value={(value ?? "") as string}
        onChange={(e) => onChange(e.target.value)}
      />
    );
  }}
/>
```

---

### RULE-FILTER-GENERATOR-BASE-02: Phải khai báo generic type cho `FilterGeneratorBase` và `FilterGeneratorBaseItem` — không dùng kiểu mặc định

`FilterGeneratorBase` nhận 2 generic type: `FieldType` (kiểu dữ liệu của field) và `ParamType` (kiểu dữ liệu của params/value). Phải khai báo cùng kiểu generic cho cả `FilterGeneratorBase` và `FilterGeneratorBaseItem` trong state.

**Sai:**

```tsx
// ❌ Không khai báo generic — TypeScript không infer được kiểu dữ liệu chính xác
const [filterItems, setFilterItems] = useState([]);

<FilterGeneratorBase
  filterItems={filterItems}
  onChangeFilterItems={setFilterItems}
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  renderField={({ value, onChange }) => null}
  renderOperators={() => null}
  renderValueInput={() => null}
/>
```

**Đúng:**

```tsx
// ✅ Khai báo generic type cho FilterGeneratorBaseItem
interface FieldOption {
  label: string;
  value: string;
}

const [filterItems, setFilterItems] = useState<FilterGeneratorBaseItem<FieldOption, string>[]>([]);

<FilterGeneratorBase<FieldOption, string>
  filterItems={filterItems}
  onChangeFilterItems={setFilterItems}
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  renderField={({ value, onChange }) => (
    <Select
      options={fieldOptions}
      value={value}
      onChange={onChange}
      fullWidth
      getLabel={(opt) => opt.label}
      getValue={(opt) => opt.value}
    />
  )}
  renderOperators={() => null}
  renderValueInput={() => null}
/>
```

---

### RULE-FILTER-GENERATOR-BASE-03: Sử dụng `renderField` để render field selector — không tự build bên ngoài

`renderField` nhận `({ value, onChange }, index)` và trả về `ReactNode`. Component tự quản lý việc khi field thay đổi sẽ reset operator và params. Không tự build field selector bên ngoài component.

**Sai:**

```tsx
// ❌ Tự build field selector bên ngoài — mất logic reset operator/params khi đổi field
<div>
  <Select
    value={selectedField}
    onChange={(field) => {
      setSelectedField(field);
      // Phải tự reset operator và params
    }}
    options={fieldOptions}
    getLabel={(opt) => opt.label}
    getValue={(opt) => opt.value}
  />
  <FilterGeneratorBase
    filterItems={filterItems}
    onChangeFilterItems={setFilterItems}
    logicType={logicType}
    onChangeLogicType={setLogicType}
    logicValue={logicValue}
    onChangeLogicValue={setLogicValue}
    renderField={() => null}
    renderOperators={() => null}
    renderValueInput={() => null}
  />
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng renderField — component tự xử lý logic reset
<FilterGeneratorBase
  filterItems={filterItems}
  onChangeFilterItems={setFilterItems}
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  renderField={({ value, onChange }) => (
    <Select
      options={fieldOptions}
      value={value}
      onChange={onChange}
      fullWidth
      clearable
      getLabel={(opt) => opt.label}
      getValue={(opt) => opt.value}
    />
  )}
  renderOperators={() => null}
  renderValueInput={() => null}
/>
```

---

### RULE-FILTER-GENERATOR-BASE-04: Sử dụng `renderOperators` để render operator dựa trên field đã chọn — render khác nhau tuỳ theo field

`renderOperators` nhận `(field, { value, onChange }, index)`. Tham số `field` là giá trị field hiện tại (hoặc `null`), cho phép render operator khác nhau tuỳ theo field đã chọn. Luôn kiểm tra `field` trước khi quyết định danh sách operator.

**Sai:**

```tsx
// ❌ Render operator cố định — không thay đổi theo field
renderOperators={(field, { value, onChange }) => (
  <Select
    value={value}
    onChange={onChange}
    options={["=", "!=", "like"]}
    getValue={(o) => o}
    getLabel={(o) => o}
    fullWidth
  />
)}
```

**Đúng:**

```tsx
// ✅ Render operator khác nhau tuỳ theo field
renderOperators={(field, { value, onChange }) => {
  let operators: string[] = [];
  if (field?.value === "name") {
    operators = ["=", "like"];
  }
  if (field?.value === "status") {
    operators = ["=", "!=", "in"];
  }

  return (
    <Select
      value={value}
      onChange={onChange}
      options={operators}
      getValue={(o) => o}
      getLabel={(o) => o}
      fullWidth
    />
  );
}}
```

---

### RULE-FILTER-GENERATOR-BASE-05: Sử dụng `renderValueInput` để render input giá trị dựa trên field và operator — kiểm tra field và op trước khi render

`renderValueInput` nhận `({ field, op }, { value, onChange }, index)`. Phải kiểm tra `field` và `op` để quyết định render kiểu input phù hợp (TextField, Select, DatePicker,...). Trả về `null` khi chưa chọn field.

**Sai:**

```tsx
// ❌ Luôn render TextField bất kể field và operator — không phù hợp với mọi trường hợp
renderValueInput={(itemState, { value, onChange }) => (
  <TextField
    fullWidth
    value={(value ?? "") as string}
    onChange={(e) => onChange(e.target.value)}
  />
)}
```

**Đúng:**

```tsx
// ✅ Kiểm tra field và op để render input phù hợp
renderValueInput={({ field, op }, { value, onChange }) => {
  if (!field) return null;

  if (op === "=" || op === "like") {
    return (
      <TextField
        fullWidth
        value={(value ?? "") as string}
        onChange={(e) => onChange(e.target.value)}
      />
    );
  }

  if (op === "in") {
    return (
      <Select
        fullWidth
        value={(value ?? null) as string | null}
        onChange={onChange}
        options={["value 1", "value 2"]}
        getLabel={(option) => option}
        getValue={(option) => option}
      />
    );
  }

  return null;
}}
```

---

### RULE-FILTER-GENERATOR-BASE-06: Sử dụng `logicType` để chọn kiểu kết hợp điều kiện — không tự implement logic kết hợp

`logicType` nhận giá trị `"AND"`, `"OR"`, `"CUSTOM"`, hoặc `"NONE"`. Component tự xử lý hiển thị giao diện tương ứng: AND/OR hiển thị nhãn giữa các điều kiện, CUSTOM hiển thị ô nhập logic tuỳ chỉnh với số thứ tự màu, NONE ẩn toàn bộ danh sách điều kiện.

**Sai:**

```tsx
// ❌ Tự implement logic kết hợp bên ngoài — trùng lặp và mất đồng bộ
const [logic, setLogic] = useState("AND");

<Select
  value={logic}
  onChange={setLogic}
  options={["AND", "OR"]}
  getValue={(o) => o}
  getLabel={(o) => o}
/>
{filterItems.map((item, index) => (
  <div key={index}>
    {index > 0 && <span>{logic}</span>}
    {/* Tự render field, operator, value */}
  </div>
))}
```

**Đúng:**

```tsx
// ✅ Sử dụng logicType — component tự xử lý hiển thị
const [logicType, setLogicType] = useState<FilterGeneratorLogicTypes>("AND");

<FilterGeneratorBase
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  filterItems={filterItems}
  onChangeFilterItems={setFilterItems}
  renderField={({ value, onChange }) => /* ... */}
  renderOperators={(field, { value, onChange }) => /* ... */}
  renderValueInput={({ field, op }, { value, onChange }) => /* ... */}
/>
```

---

### RULE-FILTER-GENERATOR-BASE-07: Sử dụng `minFilterItemCount` để giới hạn số điều kiện tối thiểu — không tự disable nút xoá

Khi truyền `minFilterItemCount`, component tự động disable nút xoá khi số lượng filter items bằng hoặc nhỏ hơn giá trị này. Không tự viết logic disable nút xoá bên ngoài.

**Sai:**

```tsx
// ❌ Tự viết logic disable nút xoá — không đồng bộ với component
<FilterGeneratorBase
  filterItems={filterItems}
  onChangeFilterItems={(items) => {
    if (items.length < 1) return; // Tự chặn xoá
    setFilterItems(items);
  }}
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  renderField={({ value, onChange }) => /* ... */}
  renderOperators={() => null}
  renderValueInput={() => null}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng minFilterItemCount — component tự disable nút xoá
<FilterGeneratorBase
  filterItems={filterItems}
  onChangeFilterItems={setFilterItems}
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  minFilterItemCount={1}
  renderField={({ value, onChange }) => /* ... */}
  renderOperators={() => null}
  renderValueInput={() => null}
/>
```

---

### RULE-FILTER-GENERATOR-BASE-08: Sử dụng `maxFilterItemCount` để giới hạn số điều kiện tối đa — không tự ẩn nút thêm

Mặc định `maxFilterItemCount=30`. Component tự động ẩn nút "Thêm điều kiện" khi đạt giới hạn. Không tự viết logic ẩn nút thêm bên ngoài.

**Sai:**

```tsx
// ❌ Tự ẩn nút thêm bên ngoài — trùng lặp logic
<FilterGeneratorBase
  filterItems={filterItems}
  onChangeFilterItems={setFilterItems}
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  renderField={({ value, onChange }) => /* ... */}
  renderOperators={() => null}
  renderValueInput={() => null}
/>
{filterItems.length < 10 && (
  <Button onClick={handleAddFilter}>Thêm điều kiện</Button>
)}
```

**Đúng:**

```tsx
// ✅ Sử dụng maxFilterItemCount — component tự ẩn nút thêm
<FilterGeneratorBase
  filterItems={filterItems}
  onChangeFilterItems={setFilterItems}
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  maxFilterItemCount={10}
  renderField={({ value, onChange }) => /* ... */}
  renderOperators={() => null}
  renderValueInput={() => null}
/>
```

---

### RULE-FILTER-GENERATOR-BASE-09: Sử dụng `getNewParamsWhenChangeOperator` để reset params khi đổi operator — không tự xử lý trong onChangeFilterItems

Khi đổi operator, có thể cần reset hoặc chuyển đổi giá trị params. Sử dụng `getNewParamsWhenChangeOperator` để trả về params mới. Trả về `undefined` để giữ nguyên params cũ. Không tự xử lý logic này trong `onChangeFilterItems`.

**Sai:**

```tsx
// ❌ Tự xử lý reset params trong onChangeFilterItems — phức tạp và dễ sai
<FilterGeneratorBase
  filterItems={filterItems}
  onChangeFilterItems={(items) => {
    const newItems = items.map((item, index) => {
      if (item.op !== filterItems[index]?.op) {
        return { ...item, params: null }; // Tự reset params
      }
      return item;
    });
    setFilterItems(newItems);
  }}
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  renderField={({ value, onChange }) => /* ... */}
  renderOperators={() => null}
  renderValueInput={() => null}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng getNewParamsWhenChangeOperator
<FilterGeneratorBase
  filterItems={filterItems}
  onChangeFilterItems={setFilterItems}
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  getNewParamsWhenChangeOperator={(op) => {
    if (op === "like") {
      return ""; // Reset về chuỗi rỗng cho operator "like"
    }
    return undefined; // Giữ nguyên params cũ
  }}
  renderField={({ value, onChange }) => /* ... */}
  renderOperators={() => null}
  renderValueInput={() => null}
/>
```

---

### RULE-FILTER-GENERATOR-BASE-10: Sử dụng `getNewOperatorWhenChangeField` để tự động đổi operator khi đổi field — không tự xử lý bên ngoài

Khi đổi field, có thể cần tự động chọn operator mới phù hợp. Sử dụng `getNewOperatorWhenChangeField` để trả về operator mới. Trả về `undefined` để giữ nguyên operator cũ. Component cũng tự động gọi `getNewParamsWhenChangeOperator` nếu operator thực sự thay đổi.

**Sai:**

```tsx
// ❌ Tự xử lý đổi operator khi đổi field — không tích hợp với flow nội bộ
renderField={({ value, onChange }) => (
  <Select
    options={fieldOptions}
    value={value}
    onChange={(field) => {
      onChange(field);
      // Tự đổi operator — không đồng bộ với component
      const newItems = filterItems.map((item) =>
        item.field === value ? { ...item, op: "=" } : item
      );
      setFilterItems(newItems);
    }}
    fullWidth
    getLabel={(opt) => opt.label}
    getValue={(opt) => opt.value}
  />
)}
```

**Đúng:**

```tsx
// ✅ Sử dụng getNewOperatorWhenChangeField
<FilterGeneratorBase
  filterItems={filterItems}
  onChangeFilterItems={setFilterItems}
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  getNewOperatorWhenChangeField={(field) => {
    if (field?.value === "status") {
      return "="; // Mặc định operator "=" cho field status
    }
    return undefined; // Giữ nguyên operator cũ
  }}
  renderField={({ value, onChange }) => (
    <Select
      options={fieldOptions}
      value={value}
      onChange={onChange}
      fullWidth
      getLabel={(opt) => opt.label}
      getValue={(opt) => opt.value}
    />
  )}
  renderOperators={() => null}
  renderValueInput={() => null}
/>
```

---

### RULE-FILTER-GENERATOR-BASE-11: Sử dụng `text` để tuỳ chỉnh nhãn cột — không override CSS hoặc tự build heading

Prop `text` nhận object `{ fieldLabel, conditionLabel, paramLabel }` để tuỳ chỉnh nhãn cho cột Field, Condition, Value trong bảng heading. Không override CSS selector nội bộ hoặc tự build heading bên ngoài.

**Sai:**

```tsx
// ❌ Tự build heading bên ngoài — trùng lặp với heading nội bộ
<div className={cx("flex items-center gap-[0.75rem]")}>
  <span>Trường dữ liệu</span>
  <span>Toán tử</span>
  <span>Giá trị</span>
</div>
<FilterGeneratorBase
  filterItems={filterItems}
  onChangeFilterItems={setFilterItems}
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  renderField={({ value, onChange }) => /* ... */}
  renderOperators={() => null}
  renderValueInput={() => null}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng text để tuỳ chỉnh nhãn cột
<FilterGeneratorBase
  filterItems={filterItems}
  onChangeFilterItems={setFilterItems}
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  text={{
    fieldLabel: "Trường dữ liệu",
    conditionLabel: "Toán tử",
    paramLabel: "Giá trị",
  }}
  renderField={({ value, onChange }) => /* ... */}
  renderOperators={() => null}
  renderValueInput={() => null}
/>
```

---

### RULE-FILTER-GENERATOR-BASE-12: Sử dụng `required` để ẩn lựa chọn "NONE" trong logic type — không tự lọc options

Khi `required=true`, lựa chọn "NONE" (Không chọn) bị ẩn khỏi danh sách logic type. Sử dụng prop này khi bắt buộc phải có logic kết hợp. Không tự lọc options bên ngoài.

**Sai:**

```tsx
// ❌ Tự chặn chọn "NONE" — không đồng bộ với logic nội bộ
<FilterGeneratorBase
  filterItems={filterItems}
  onChangeFilterItems={setFilterItems}
  logicType={logicType}
  onChangeLogicType={(type) => {
    if (type === "NONE") return; // Tự chặn
    setLogicType(type);
  }}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  renderField={({ value, onChange }) => /* ... */}
  renderOperators={() => null}
  renderValueInput={() => null}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng required — component tự ẩn option "NONE"
<FilterGeneratorBase
  required
  filterItems={filterItems}
  onChangeFilterItems={setFilterItems}
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  renderField={({ value, onChange }) => /* ... */}
  renderOperators={() => null}
  renderValueInput={() => null}
/>
```

---

### RULE-FILTER-GENERATOR-BASE-13: Sử dụng `disabled` để vô hiệu hoá toàn bộ bộ lọc — không tự ẩn giao diện

Khi `disabled=true`, Select logic type bị disable và toàn bộ danh sách filter items bị ẩn. Sử dụng prop này khi cần vô hiệu hoá bộ lọc. Không tự render có điều kiện component.

**Sai:**

```tsx
// ❌ Tự ẩn component — không giữ lại logic type select
{!isDisabled && (
  <FilterGeneratorBase
    filterItems={filterItems}
    onChangeFilterItems={setFilterItems}
    logicType={logicType}
    onChangeLogicType={setLogicType}
    logicValue={logicValue}
    onChangeLogicValue={setLogicValue}
    renderField={({ value, onChange }) => /* ... */}
    renderOperators={() => null}
    renderValueInput={() => null}
  />
)}
```

**Đúng:**

```tsx
// ✅ Sử dụng disabled — component tự xử lý hiển thị
<FilterGeneratorBase
  disabled={isDisabled}
  filterItems={filterItems}
  onChangeFilterItems={setFilterItems}
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  renderField={({ value, onChange }) => /* ... */}
  renderOperators={() => null}
  renderValueInput={() => null}
/>
```

---

### RULE-FILTER-GENERATOR-BASE-14: Sử dụng `showCustomLogicError` để kiểm soát hiển thị lỗi custom logic — không tự validate bên ngoài

Khi `logicType="CUSTOM"`, component tự validate biểu thức logic (kiểm tra dấu ngoặc, ký tự hợp lệ, điều kiện tồn tại). Mặc định `showCustomLogicError=true` hiển thị lỗi. Truyền `showCustomLogicError={false}` khi muốn tự kiểm soát thời điểm hiển thị lỗi.

**Sai:**

```tsx
// ❌ Tự validate biểu thức logic bên ngoài — trùng lặp với validation nội bộ
const [customError, setCustomError] = useState("");

const handleLogicChange = (value: string) => {
  setLogicValue(value);
  // Tự validate — logic đã có sẵn trong component
  if (!value.match(/^[\d\s()ANDOR]+$/i)) {
    setCustomError("Biểu thức không hợp lệ");
  }
};
```

**Đúng:**

```tsx
// ✅ Component tự validate — dùng showCustomLogicError để kiểm soát hiển thị
<FilterGeneratorBase
  filterItems={filterItems}
  onChangeFilterItems={setFilterItems}
  logicType={logicType}
  onChangeLogicType={setLogicType}
  logicValue={logicValue}
  onChangeLogicValue={setLogicValue}
  showCustomLogicError={true}
  renderField={({ value, onChange }) => /* ... */}
  renderOperators={() => null}
  renderValueInput={() => null}
/>
```

---

### Bảng giá trị mặc định

| Prop | Giá trị mặc định | Mô tả |
|---|---|---|
| `required` | `false` | Khi `true`, ẩn lựa chọn "NONE" trong logic type |
| `disabled` | `false` | Khi `true`, disable logic type select và ẩn danh sách filter items |
| `showCustomLogicError` | `true` | Khi `true`, hiển thị lỗi validation cho custom logic |
| `maxFilterItemCount` | `30` | Số lượng filter items tối đa, ẩn nút thêm khi đạt giới hạn |
| `title` | `"Logic kết hợp điều kiện"` (i18n) | Tiêu đề hiển thị phía trên Select logic type |
| `text.fieldLabel` | `"Trường"` (i18n) | Nhãn cột field trong heading |
| `text.conditionLabel` | `"Điều kiện"` (i18n) | Nhãn cột condition trong heading |
| `text.paramLabel` | `"Giá trị"` (i18n) | Nhãn cột value trong heading |
