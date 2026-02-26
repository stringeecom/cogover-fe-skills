---
title: Cach su dung component FilterGenerator trong ui-kit
impact: HIGH
impactDescription: Su dung sai dan den mat dieu kien loc, logic ket hop khong hoat dong, loi TypeScript, va hanh vi filter khong nhat quan
tags: filter, filter-generator, condition, logic, datafield, operator, ui-kit
---

## Cach su dung component FilterGenerator trong ui-kit

Duong dan component: `ui-kit/src/components/FilterGenerator`

Props bat buoc: `dataFields`, `logicType`, `logicValue`, `filterItems`, `onChangeLogicType`, `onChangeLogicValue`, `onChangeFilterItems`.
Props tuy chon: `recordDataFields`, `required`, `disabled`, `readOnly`, `clearable`, `maxFilterItemCount`, `showCustomLogicError`, `title`, `text`, `showPreview`, `onClickPreview`, `disabledPreviewButton`, `onReset`, `renderLabel`, `renderValueInputWrapper`, `insertFieldObjectButtonWrapperProps`, `onChangeFilterItemParams`, `isWorkflow`.

Ke thua tu `React.HTMLAttributes<HTMLDivElement>` nen ho tro: `className`, `style`, `id`, `onClick`, `onMouseEnter`, va cac thuoc tinh HTML div khac.

Gia tri mac dinh: `required=false`, `disabled=false`, `readOnly=false`, `showCustomLogicError=true`, `showPreview=false`, `disabledPreviewButton=false`, `maxFilterItemCount=30`, `isWorkflow=false`.

---

### RULE-FILTER-GENERATOR-01: FilterGenerator la controlled component — phai truyen day du `logicType`, `logicValue`, `filterItems` va cac ham onChange tuong ung

`FilterGenerator` la controlled component. Phai truyen `logicType` (kieu logic ket hop: `"AND"`, `"OR"`, `"CUSTOM"`, `"NONE"`), `logicValue` (chuoi bieu thuc logic tuy chinh), `filterItems` (mang cac dieu kien loc), va 3 ham `onChangeLogicType`, `onChangeLogicValue`, `onChangeFilterItems` de cap nhat state. Khong tu quan ly state ben trong component.

**Sai:**

```tsx
// ❌ Thieu cac ham onChange — component khong hoat dong
<FilterGenerator
  dataFields={dataFields}
  logicType="AND"
  logicValue=""
  filterItems={[]}
/>
```

**Dung:**

```tsx
// ✅ Controlled component voi day du state va onChange handlers
const [logicType, setLogicType] = useState<FilterGeneratorLogicTypes>("AND");
const [logicValue, setLogicValue] = useState("");
const [filterItems, setFilterItems] = useState<FilterGeneratorItem[]>([]);

<FilterGenerator
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

---

### RULE-FILTER-GENERATOR-02: `dataFields` phai la mang `DataField[]` hop le — khong truyen mang rong hoac du lieu khong dung kieu

`dataFields` la mang cac truong du lieu (DataField) de hien thi trong dropdown chon truong. Component tu dong loc cac truong co `status === DataFieldStatus.Active` va xu ly truong `formula` dac biet. Phai truyen du lieu tu API hoac state hop le.

**Sai:**

```tsx
// ❌ Truyen mang doi tuong khong dung kieu DataField
<FilterGenerator
  dataFields={[{ name: "Ten", type: "text" }]}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

**Dung:**

```tsx
// ✅ Truyen dataFields tu API voi kieu DataField[] hop le
const { data: object } = useQueryObjectDetailBySlug({ slug: objectSlug });

<FilterGenerator
  dataFields={object?.fields ?? []}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

---

### RULE-FILTER-GENERATOR-03: Su dung `logicType` de chon kieu logic ket hop — khong tu build logic select ben ngoai

Component tich hop san Select cho phep chon kieu logic: `"AND"` (tat ca dieu kien), `"OR"` (mot trong cac dieu kien), `"CUSTOM"` (logic tuy chinh), `"NONE"` (khong chon). Khi `logicType="CUSTOM"`, component tu dong hien thi input de nhap bieu thuc logic. Khong tu build dropdown hoac input logic ben ngoai.

**Sai:**

```tsx
// ❌ Tu build logic select ben ngoai — mat dong bo voi logic noi bo
<Select
  value={logicType}
  onChange={setLogicType}
  options={["AND", "OR", "CUSTOM"]}
/>
<FilterGenerator
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

**Dung:**

```tsx
// ✅ Component tu hien thi logic select — chi can truyen state va onChange
<FilterGenerator
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

---

### RULE-FILTER-GENERATOR-04: Su dung `logicType="CUSTOM"` ket hop `logicValue` de tao bieu thuc logic tuy chinh — khong tu tinh toan logic

Khi `logicType="CUSTOM"` va co nhieu hon 1 dieu kien, component hien thi input de nhap bieu thuc logic (vi du: `"1 AND (2 OR 3)"`). Component tu dong validate bieu thuc va hien thi loi neu khong hop le. Khong tu tinh toan logic ket hop ben ngoai.

**Sai:**

```tsx
// ❌ Tu tinh toan logic ket hop — trung lap voi logic noi bo
const customLogic = filterItems
  .map((_, i) => i + 1)
  .join(" AND ");

<FilterGenerator
  dataFields={dataFields}
  logicType="AND"
  logicValue={customLogic}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

**Dung:**

```tsx
// ✅ Su dung logicType="CUSTOM" — component tu hien thi input va validate
const [logicType, setLogicType] = useState<FilterGeneratorLogicTypes>("CUSTOM");
const [logicValue, setLogicValue] = useState("1 AND (2 OR 3)");

<FilterGenerator
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

---

### RULE-FILTER-GENERATOR-05: Su dung `required` de an tuy chon "NONE" trong logic select — khong tu loc options

Khi `required=true`, tuy chon `"NONE"` (khong chon) se bi an khoi dropdown logic type. Mac dinh `required=false` cho phep nguoi dung chon `"NONE"` de bo filter. Khong tu loc logic options ben ngoai.

**Sai:**

```tsx
// ❌ Tu loc logic options ben ngoai — mat dong bo voi component
{logicType !== "NONE" && (
  <FilterGenerator
    dataFields={dataFields}
    logicType={logicType}
    logicValue={logicValue}
    filterItems={filterItems}
    onChangeLogicType={setLogicType}
    onChangeLogicValue={setLogicValue}
    onChangeFilterItems={setFilterItems}
  />
)}
```

**Dung:**

```tsx
// ✅ Su dung required de bat buoc chon logic type
<FilterGenerator
  required
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

---

### RULE-FILTER-GENERATOR-06: Su dung `recordDataFields` ket hop `text` de hien thi truong du lieu record theo nhom — khong tu gop danh sach

Khi truyen `recordDataFields`, component tu dong hien thi danh sach truong theo 2 nhom (object fields va record fields) trong dropdown Select voi `mode="group"`. Prop `text.objectDataFieldLabel` va `text.recordDataFieldLabel` tuy chinh ten nhom. Khong tu gop 2 mang dataFields va recordDataFields lai.

**Sai:**

```tsx
// ❌ Tu gop 2 mang — mat phan nhom va prefix "$record."
<FilterGenerator
  dataFields={[...objectFields, ...recordFields]}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

**Dung:**

```tsx
// ✅ Truyen recordDataFields rieng — component tu dong nhom va them prefix
<FilterGenerator
  dataFields={objectFields}
  recordDataFields={recordFields}
  text={{
    objectDataFieldLabel: "Truong doi tuong",
    recordDataFieldLabel: "Truong ban ghi",
  }}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

---

### RULE-FILTER-GENERATOR-07: Su dung `clearable` ket hop `onReset` de cho phep reset filter — khong tu build nut reset

Khi `clearable=true` va khong phai `readOnly`, component hien thi nut "Reset" khi co it nhat 1 dieu kien loc. Click nut reset se goi `onReset()`, xoa tat ca filterItems, dat logicType ve `"AND"` va logicValue ve `""`. Khong tu build nut reset ben ngoai.

**Sai:**

```tsx
// ❌ Tu build nut reset ben ngoai — trung lap va khong dong bo
<div>
  <FilterGenerator
    dataFields={dataFields}
    logicType={logicType}
    logicValue={logicValue}
    filterItems={filterItems}
    onChangeLogicType={setLogicType}
    onChangeLogicValue={setLogicValue}
    onChangeFilterItems={setFilterItems}
  />
  <button
    onClick={() => {
      setFilterItems([]);
      setLogicType("AND");
      setLogicValue("");
    }}
  >
    Reset
  </button>
</div>
```

**Dung:**

```tsx
// ✅ Su dung clearable va onReset — component tu hien thi nut reset
<FilterGenerator
  clearable
  onReset={() => {
    // Xu ly them khi reset (vi du: goi API)
  }}
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

---

### RULE-FILTER-GENERATOR-08: `disabled` va `readOnly` co hanh vi khac nhau — khong nham lan

`disabled=true` lam an toan bo danh sach dieu kien loc va vo hieu hoa Select logic type. `readOnly=true` van hien thi danh sach dieu kien loc nhung khong cho chinh sua (Select disabled, an nut xoa va nut them dieu kien). Dung `readOnly` khi chi muon hien thi, dung `disabled` khi muon an hoan toan.

**Sai:**

```tsx
// ❌ Dung disabled khi chi muon khong cho chinh sua — mat hien thi danh sach dieu kien
<FilterGenerator
  disabled
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

**Dung:**

```tsx
// ✅ Dung readOnly de hien thi dieu kien nhung khong cho chinh sua
<FilterGenerator
  readOnly
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>

// ✅ Dung disabled khi muon an hoan toan danh sach dieu kien
<FilterGenerator
  disabled
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

---

### RULE-FILTER-GENERATOR-09: Su dung `maxFilterItemCount` de gioi han so dieu kien — khong tu gioi han ben ngoai

Mac dinh `maxFilterItemCount=30`. Khi so luong `filterItems` dat toi da, nut "Them dieu kien" tu dong an. Truyen gia tri khac neu can gioi han it hoac nhieu hon. Khong tu an nut hoac chan logic them dieu kien ben ngoai.

**Sai:**

```tsx
// ❌ Tu gioi han so dieu kien — trung lap voi logic noi bo
<FilterGenerator
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={(items) => {
    if (items.length <= 10) {
      setFilterItems(items);
    }
  }}
/>
```

**Dung:**

```tsx
// ✅ Su dung maxFilterItemCount de gioi han
<FilterGenerator
  maxFilterItemCount={10}
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

---

### RULE-FILTER-GENERATOR-10: Su dung `showPreview` va `onClickPreview` de hien thi nut xem truoc — khong tu build nut preview

Khi `showPreview=true`, component hien thi nut "Xem truoc ban ghi da loc" ben canh nut reset. Dung `onClickPreview` de xu ly su kien click va `disabledPreviewButton` de vo hieu hoa nut. Khong tu build nut preview ben ngoai.

**Sai:**

```tsx
// ❌ Tu build nut preview ben ngoai — khong dong bo vi tri voi layout component
<FilterGenerator
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
<Button onClick={handlePreview}>Xem truoc</Button>
```

**Dung:**

```tsx
// ✅ Su dung showPreview va onClickPreview
<FilterGenerator
  showPreview
  onClickPreview={handlePreview}
  disabledPreviewButton={isLoading}
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

---

### RULE-FILTER-GENERATOR-11: Su dung `insertFieldObjectButtonWrapperProps` khi can chon gia tri tu bien — khong tu build variable picker

Khi can chon gia tri tu bien (vi du: `$currentRecord`), truyen `insertFieldObjectButtonWrapperProps` voi `rootObjects` chua danh sach doi tuong co the chon. Component tu dong hien thi nut chon bien trong phan nhap gia tri cua moi dieu kien loc.

**Sai:**

```tsx
// ❌ Tu build variable picker ben ngoai — khong tich hop voi FilterParamsInput
<div>
  <VariablePicker onSelect={handleSelectVariable} />
  <FilterGenerator
    dataFields={dataFields}
    logicType={logicType}
    logicValue={logicValue}
    filterItems={filterItems}
    onChangeLogicType={setLogicType}
    onChangeLogicValue={setLogicValue}
    onChangeFilterItems={setFilterItems}
  />
</div>
```

**Dung:**

```tsx
// ✅ Su dung insertFieldObjectButtonWrapperProps
<FilterGenerator
  isWorkflow
  insertFieldObjectButtonWrapperProps={{
    rootObjects: [{ slug: "personnel", variableSlug: "$currentRecord" }],
  }}
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

---

### RULE-FILTER-GENERATOR-12: Su dung `renderValueInputWrapper` de tuy chinh wrapper cua input gia tri — khong override CSS noi bo

`renderValueInputWrapper` nhan `filterItem`, `onChangeFilterItem`, va `children` (FilterParamsInput goc). Dung de boc them logic hoac UI xung quanh input gia tri cua moi dieu kien. Khong override CSS selector noi bo.

**Sai:**

```tsx
// ❌ Override CSS noi bo — de vo khi component cap nhat
<FilterGenerator
  className={cx("[&_.stringee-filter-generator_input]:border-red-500")}
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

**Dung:**

```tsx
// ✅ Su dung renderValueInputWrapper de tuy chinh
<FilterGenerator
  renderValueInputWrapper={({ filterItem, onChangeFilterItem, children }) => (
    <div className={cx("flex items-center gap-[0.5rem]")}>
      {children}
      <Tooltip content={`Field: ${filterItem.field}`}>
        <InfoIcon />
      </Tooltip>
    </div>
  )}
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

---

### RULE-FILTER-GENERATOR-13: Su dung `showCustomLogicError` de kiem soat hien thi loi logic tuy chinh — khong tu validate

Mac dinh `showCustomLogicError=true` — component tu dong validate va hien thi loi khi bieu thuc logic khong hop le (sai cu phap, dieu kien khong ton tai, ngoac khong dung). Truyen `showCustomLogicError={false}` khi muon tu xu ly hien thi loi. Khong tu validate bieu thuc logic ben ngoai.

**Sai:**

```tsx
// ❌ Tu validate logic expression — trung lap voi logic noi bo
const isValidLogic = validateExpression(logicValue, filterItems.length);

<FilterGenerator
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
{!isValidLogic && <p className={cx("text-error")}>Logic khong hop le</p>}
```

**Dung:**

```tsx
// ✅ De component tu validate va hien thi loi
<FilterGenerator
  showCustomLogicError
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>

// ✅ Hoac tat loi mac dinh khi muon tu xu ly hien thi
<FilterGenerator
  showCustomLogicError={false}
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

---

### RULE-FILTER-GENERATOR-14: Su dung `FilterGeneratorItem` dung cau truc khi khoi tao filterItems — khong tu dinh nghia kieu

Moi `FilterGeneratorItem` phai co `field` (slug cua truong hoac null), `op` (Operator hoac null), `params` (gia tri loc), `fieldType` (kieu truong du lieu). Truong `iu` (is current user value) la optional, dung khi gia tri la bien nguoi dung hien tai. Khong tu dinh nghia kieu filter item.

**Sai:**

```tsx
// ❌ Tu dinh nghia kieu filter item — khong tuong thich voi component
const [filterItems, setFilterItems] = useState([
  { column: "name", operator: "equals", value: "test" },
]);
```

**Dung:**

```tsx
// ✅ Su dung dung cau truc FilterGeneratorItem
const [filterItems, setFilterItems] = useState<FilterGeneratorItem[]>([
  {
    field: "created_by",
    op: "=",
    params: "$currentRecord",
    fieldType: "lookup_normal",
    iu: false,
  },
  {
    field: "first_last_name",
    op: "not null",
    params: null,
    fieldType: "formula",
  },
]);
```

---

### RULE-FILTER-GENERATOR-15: Su dung `onChangeFilterItemParams` khi can theo doi thay doi params cua tung dieu kien — khong tu so sanh filterItems

`onChangeFilterItemParams` nhan `params`, `index`, va `filterItem` moi khi gia tri params cua mot dieu kien thay doi. Dung de xu ly logic phu (vi du: goi API khi gia tri thay doi) ma khong can so sanh mang `filterItems` cu va moi.

**Sai:**

```tsx
// ❌ Tu so sanh filterItems de phat hien thay doi params — phuc tap va de loi
useEffect(() => {
  const changedIndex = filterItems.findIndex(
    (item, i) => item.params !== prevFilterItems[i]?.params,
  );
  if (changedIndex >= 0) {
    handleParamsChange(filterItems[changedIndex].params, changedIndex);
  }
}, [filterItems]);
```

**Dung:**

```tsx
// ✅ Su dung onChangeFilterItemParams
<FilterGenerator
  onChangeFilterItemParams={(params, index, filterItem) => {
    handleParamsChange(params, index);
  }}
  dataFields={dataFields}
  logicType={logicType}
  logicValue={logicValue}
  filterItems={filterItems}
  onChangeLogicType={setLogicType}
  onChangeLogicValue={setLogicValue}
  onChangeFilterItems={setFilterItems}
/>
```

---

### Bang gia tri mac dinh

| Prop | Gia tri mac dinh | Mo ta |
|---|---|---|
| `required` | `false` | Khi `true`, an tuy chon "NONE" trong logic select |
| `disabled` | `false` | Khi `true`, an toan bo danh sach dieu kien va vo hieu hoa logic select |
| `readOnly` | `false` | Khi `true`, hien thi dieu kien nhung khong cho chinh sua |
| `showCustomLogicError` | `true` | Khi `true`, hien thi loi validation cho logic tuy chinh |
| `maxFilterItemCount` | `30` | So luong dieu kien loc toi da |
| `showPreview` | `false` | Khi `true`, hien thi nut xem truoc ban ghi da loc |
| `disabledPreviewButton` | `false` | Khi `true`, vo hieu hoa nut xem truoc |
| `isWorkflow` | `false` | Khi `true`, bat che do workflow (anh huong den logic operator va variable) |
| `clearable` | `undefined` | Khi `true`, hien thi nut reset filter |
| `title` | `t("filter:createFilter.logicCombineConditions")` | Tieu de hien thi phia tren logic select |
