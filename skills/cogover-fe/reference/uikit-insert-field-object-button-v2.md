---
title: Cach su dung component InsertFieldObjectButtonV2 trong ui-kit
impact: HIGH
impactDescription: Su dung sai dan den khong mo duoc modal chon truong du lieu, mat gia tri da chon, loi TypeScript ve kieu du lieu rootObjects, va khong loc duoc truong theo dieu kien
tags: insert-field, data-field, variable, cascader, modal, object, ui-kit
---

## Cach su dung component InsertFieldObjectButtonV2 trong ui-kit

Duong dan component: `ui-kit/src/components/InsertFieldObjectButtonV2`

Props bat buoc: `rootObjects`, `onInsert`.
Props tuy chon: `isDisabled`, `children`, `checkSelectableField`, `selectedDataField`, `maxLevel`, `description`, `filterDataField`.

Component nay la mot nut bam mo modal cho phep nguoi dung chon truong du lieu (data field) tu danh sach cac object. Khi nguoi dung chon xong, callback `onInsert` se tra ve variable slug va tree data cua truong duoc chon.

Component ke thua props tu `InsertFieldObjectV2Props` (ngoai tru `object` va `onClose`) nen ho tro: `rootObjects`, `onInsert`, `checkSelectableField`, `selectedDataField`, `maxLevel`, `description`, `filterDataField`.

Gia tri mac dinh: `isDisabled=false`, `maxLevel=5`, `selectedDataField=null`, `filterDataField=() => true`.

---

### RULE-INSERT-FIELD-V2-01: Phai truyen `rootObjects` va `onInsert` — day la cac props bat buoc

`InsertFieldObjectButtonV2` yeu cau `rootObjects` (mang cac object goc) va `onInsert` (callback nhan ve variable slug va tree data khi nguoi dung chon). Thieu cac props nay se khien modal khong hien thi du lieu hoac khong tra ve gia tri.

**Sai:**

```tsx
// ❌ Thieu rootObjects va onInsert — modal khong co du lieu va khong tra ve ket qua
<InsertFieldObjectButtonV2 />
```

**Dung:**

```tsx
// ✅ Truyen rootObjects va onInsert day du
const handleInsert = (value: string, treeData: TreeFieldObjectV2) => {
  console.log("123:variable slug", value);
};

<InsertFieldObjectButtonV2
  rootObjects={[
    {
      slug: "contact",
      variableSlug: "$contact",
    },
  ]}
  onInsert={handleInsert}
/>
```

---

### RULE-INSERT-FIELD-V2-02: Cau hinh `rootObjects` dung dinh dang `InsertFieldRootObjectItem` — khong truyen sai kieu du lieu

Moi item trong `rootObjects` phai co `slug` (slug cua object) va `variableSlug` (bien so duoc tao khi chon). Tuy chon co `isAdditionalOption` (de tao option khong phai object nhu `$currentTime`) va `renderName` (de tuy chinh hien thi ten). Khong truyen sai dinh dang.

**Sai:**

```tsx
// ❌ Truyen sai dinh dang — thieu variableSlug
<InsertFieldObjectButtonV2
  rootObjects={[
    {
      slug: "contact",
      name: "Lien he",
    },
  ]}
  onInsert={handleInsert}
/>
```

**Dung:**

```tsx
// ✅ Truyen dung dinh dang InsertFieldRootObjectItem
<InsertFieldObjectButtonV2
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
  onInsert={handleInsert}
/>
```

---

### RULE-INSERT-FIELD-V2-03: Su dung `isAdditionalOption` cho cac option dac biet khong phai object — khong tu tao option gia

Khi can them option dac biet nhu `$currentTime` hoac `$currentUser` khong phai la object trong he thong, su dung `isAdditionalOption: true` trong `rootObjects`. Option nay se luon hien thi trong danh sach va nguoi dung co the chon truc tiep ma khong can duyet vao children. Khong tu tao option gia bang cach them vao danh sach object.

**Sai:**

```tsx
// ❌ Tu tao object gia de them option dac biet — se bi loi khi query API
<InsertFieldObjectButtonV2
  rootObjects={[
    {
      slug: "contact",
      variableSlug: "$contact",
    },
    {
      slug: "currentTime",
      variableSlug: "$currentTime",
    },
  ]}
  onInsert={handleInsert}
/>
```

**Dung:**

```tsx
// ✅ Su dung isAdditionalOption cho option dac biet
<InsertFieldObjectButtonV2
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
  onInsert={handleInsert}
/>
```

---

### RULE-INSERT-FIELD-V2-04: Su dung `children` render prop de tuy chinh nut bam — khong boc component ben ngoai

Component ho tro `children` la mot render prop nhan `{ onClick }`. Mac dinh se render `IconButton` voi icon `BracketsCurlyIcon`. Khi can tuy chinh nut bam, su dung `children` render prop. Khong tu boc component ben ngoai de xu ly viec mo modal.

**Sai:**

```tsx
// ❌ Tu boc component ben ngoai — khong dong bo voi state mo/dong modal noi bo
const [open, setOpen] = useState(false);

<div>
  <Button onClick={() => setOpen(true)}>Chen truong</Button>
  {open && (
    <InsertFieldObjectV2
      rootObjects={rootObjects}
      onInsert={handleInsert}
      onClose={() => setOpen(false)}
    />
  )}
</div>
```

**Dung:**

```tsx
// ✅ Su dung children render prop de tuy chinh nut bam
<InsertFieldObjectButtonV2
  rootObjects={rootObjects}
  onInsert={handleInsert}
>
  {({ onClick }) => (
    <Button variant="outlined" onClick={onClick}>
      Chen truong du lieu
    </Button>
  )}
</InsertFieldObjectButtonV2>
```

---

### RULE-INSERT-FIELD-V2-05: Su dung `isDisabled` de vo hieu hoa nut — khong tu them prop disabled vao children

Component co prop `isDisabled` de vo hieu hoa nut bam mac dinh. Khi su dung children tuy chinh, `isDisabled` khong tu dong truyen vao — phai tu xu ly trong render prop. Khong tu them state disabled ben ngoai khi su dung nut mac dinh.

**Sai:**

```tsx
// ❌ Tu boc div de chan click — khong can thiet va pha vo layout
<div style={{ pointerEvents: "none" }}>
  <InsertFieldObjectButtonV2
    rootObjects={rootObjects}
    onInsert={handleInsert}
  />
</div>
```

**Dung:**

```tsx
// ✅ Su dung isDisabled cho nut mac dinh
<InsertFieldObjectButtonV2
  isDisabled
  rootObjects={rootObjects}
  onInsert={handleInsert}
/>

// ✅ Tu xu ly disabled khi su dung children tuy chinh
<InsertFieldObjectButtonV2
  isDisabled={isFormDisabled}
  rootObjects={rootObjects}
  onInsert={handleInsert}
>
  {({ onClick }) => (
    <Button
      variant="outlined"
      disabled={isFormDisabled}
      onClick={onClick}
    >
      Chen truong
    </Button>
  )}
</InsertFieldObjectButtonV2>
```

---

### RULE-INSERT-FIELD-V2-06: Su dung `selectedDataField` de tu dong loc truong tuong thich — khong tu viet logic loc

Khi truyen `selectedDataField`, component se su dung ham `defaultFilterSelectableField` de tu dong loc cac truong tuong thich (cung fieldType, cung multiple, xu ly formula va lookup). Chi cac truong tuong thich moi cho phep chon. Khong tu viet logic loc ben ngoai khi co the su dung prop nay.

**Sai:**

```tsx
// ❌ Tu viet logic loc bang checkSelectableField khi chi can loc theo kieu — trung lap voi logic mac dinh
<InsertFieldObjectButtonV2
  rootObjects={rootObjects}
  onInsert={handleInsert}
  checkSelectableField={(field) => {
    return field.fieldType === emailField.fieldType && field.multiple === emailField.multiple;
  }}
/>
```

**Dung:**

```tsx
// ✅ Truyen selectedDataField — component tu dong loc truong tuong thich
<InsertFieldObjectButtonV2
  rootObjects={rootObjects}
  onInsert={handleInsert}
  selectedDataField={emailField}
/>
```

---

### RULE-INSERT-FIELD-V2-07: Su dung `checkSelectableField` khi can logic loc phuc tap hon mac dinh

Khi logic loc mac dinh (dua tren `selectedDataField`) khong du, su dung `checkSelectableField` de tuy chinh. Ham nay nhan vao `DataField` va tra ve `boolean` hoac `{ selectable: boolean; hide: boolean }`. Tra ve `{ selectable: false, hide: true }` de an hoan toan truong khoi danh sach.

**Sai:**

```tsx
// ❌ Su dung filterDataField de an truong khong cho chon — filterDataField chi loc truong hien thi, khong kiem soat selectable
<InsertFieldObjectButtonV2
  rootObjects={rootObjects}
  onInsert={handleInsert}
  filterDataField={(field) => field.fieldType !== "formula"}
/>
```

**Dung:**

```tsx
// ✅ Su dung checkSelectableField de kiem soat ca selectable va hien thi
<InsertFieldObjectButtonV2
  rootObjects={rootObjects}
  onInsert={handleInsert}
  checkSelectableField={(field) => {
    if (field.fieldType === "formula") {
      return { selectable: false, hide: true };
    }
    return field.fieldType === "short_text" || field.fieldType === "long_text";
  }}
/>
```

---

### RULE-INSERT-FIELD-V2-08: Su dung `filterDataField` de loc danh sach truong hien thi — khong nham lan voi `checkSelectableField`

`filterDataField` loc truong truoc khi render vao tree (truong bi loc se khong hien thi). `checkSelectableField` kiem soat truong nao duoc phep chon (truong khong selectable van hien thi nhung khong click duoc). Su dung dung prop cho dung muc dich.

**Sai:**

```tsx
// ❌ Dung checkSelectableField de an truong — truong van co the hien thi neu la lookup
<InsertFieldObjectButtonV2
  rootObjects={rootObjects}
  onInsert={handleInsert}
  checkSelectableField={(field) => field.fieldType !== "attachment"}
/>
```

**Dung:**

```tsx
// ✅ Dung filterDataField de loai bo hoan toan truong khong muon hien thi
<InsertFieldObjectButtonV2
  rootObjects={rootObjects}
  onInsert={handleInsert}
  filterDataField={(field) => field.fieldType !== "attachment"}
/>
```

---

### RULE-INSERT-FIELD-V2-09: Su dung `maxLevel` de gioi han do sau duyet object lien ket — khong de mac dinh khi cau truc don gian

Mac dinh `maxLevel=5` cho phep duyet sau 5 cap object lien ket (lookup). Khi cau truc du lieu don gian chi can chon truong cap 1, truyen `maxLevel={1}` de tranh nguoi dung duyet qua sau. Khong de mac dinh khi khong can thiet.

**Sai:**

```tsx
// ❌ De mac dinh maxLevel=5 khi chi can chon truong cap 1 — nguoi dung co the duyet qua sau gay nham lan
<InsertFieldObjectButtonV2
  rootObjects={rootObjects}
  onInsert={handleInsert}
/>
```

**Dung:**

```tsx
// ✅ Gioi han maxLevel khi cau truc don gian
<InsertFieldObjectButtonV2
  rootObjects={rootObjects}
  onInsert={handleInsert}
  maxLevel={1}
/>

// ✅ De mac dinh khi can duyet nhieu cap object lien ket
<InsertFieldObjectButtonV2
  rootObjects={rootObjects}
  onInsert={handleInsert}
/>
```

---

### RULE-INSERT-FIELD-V2-10: Su dung `description` de thay doi mo ta trong modal — khong override CSS noi bo

Modal hien thi mo ta phia tren danh sach chon truong. Mac dinh la text "Chon truong du lieu de chen". Khi can thay doi mo ta, truyen prop `description`. Khong override CSS noi bo de them text.

**Sai:**

```tsx
// ❌ Override CSS noi bo de them mo ta — de vo khi component cap nhat
<InsertFieldObjectButtonV2
  className={cx("[&_.stringee-modal-body]:before:content-['Chon_bien']")}
  rootObjects={rootObjects}
  onInsert={handleInsert}
/>
```

**Dung:**

```tsx
// ✅ Su dung prop description
<InsertFieldObjectButtonV2
  rootObjects={rootObjects}
  onInsert={handleInsert}
  description="Chon truong du lieu tuong thich de chen vao cong thuc"
/>

// ✅ Truyen ReactNode cho description phuc tap
<InsertFieldObjectButtonV2
  rootObjects={rootObjects}
  onInsert={handleInsert}
  description={
    <span className={cx("text-typo-secondary")}>
      Chi hien thi cac truong kieu <strong>text</strong>
    </span>
  }
/>
```

---

### RULE-INSERT-FIELD-V2-11: Su dung `renderName` trong rootObjects de tuy chinh ten hien thi — khong tu tao tree data

Khi can tuy chinh ten hien thi cua root object trong cascader, su dung `renderName` trong `InsertFieldRootObjectItem`. Ham nay nhan `ObjectListResponse | null` va tra ve `ReactNode`. Khong tu tao tree data de thay doi ten hien thi.

**Sai:**

```tsx
// ❌ Tu tao tree data de thay doi ten — khong can thiet va mat dong bo voi API
const customTreeData = objectList.map((obj) => ({
  ...obj,
  name: `Custom: ${obj.name}`,
}));
```

**Dung:**

```tsx
// ✅ Su dung renderName trong rootObjects
<InsertFieldObjectButtonV2
  rootObjects={[
    {
      slug: "contact",
      variableSlug: "$contact",
      renderName: (object) => object?.name ?? "$contact",
    },
    {
      slug: "currentTime",
      variableSlug: "$currentTime",
      isAdditionalOption: true,
      renderName: () => "Thoi gian hien tai",
    },
  ]}
  onInsert={handleInsert}
/>
```

---

### RULE-INSERT-FIELD-V2-12: Ket hop voi `InsertFieldObjectButtonWrapper2` khi can hien thi gia tri da chon va nut xoa

Khi can hien thi gia tri variable slug da chon ben canh input va co nut xoa, su dung `InsertFieldObjectButtonWrapper2` thay vi tu build. Component nay boc `InsertFieldObjectButtonV2` va them logic hien thi gia tri, tooltip va nut clear.

**Sai:**

```tsx
// ❌ Tu build wrapper — trung lap logic va thieu tinh nang nhu tooltip, render personnel value
const [selectedVariable, setSelectedVariable] = useState<string | null>(null);

<div className={cx("flex items-center")}>
  {selectedVariable ? (
    <div className={cx("flex items-center gap-[0.5rem]")}>
      <span>{selectedVariable}</span>
      <button onClick={() => setSelectedVariable(null)}>X</button>
    </div>
  ) : (
    <TextField />
  )}
  <InsertFieldObjectButtonV2
    rootObjects={rootObjects}
    onInsert={(value) => setSelectedVariable(value)}
  />
</div>
```

**Dung:**

```tsx
// ✅ Su dung InsertFieldObjectButtonWrapper2
const [selectedVariable, setSelectedVariable] = useState<string | null>(null);

<InsertFieldObjectButtonWrapper2
  selectedVariableSlug={selectedVariable}
  selectedDataField={emailField}
  onClear={() => setSelectedVariable(null)}
  onInsert={(value) => setSelectedVariable(value)}
  rootObjects={[
    {
      slug: "contact",
      variableSlug: "$contact",
    },
  ]}
>
  <TextField />
</InsertFieldObjectButtonWrapper2>
```

---

| Prop | Gia tri mac dinh | Mo ta |
|---|---|---|
| `isDisabled` | `false` | Vo hieu hoa nut bam |
| `children` | `IconButton` voi `BracketsCurlyIcon` | Render prop tuy chinh nut bam |
| `maxLevel` | `5` | Do sau toi da duyet object lien ket |
| `selectedDataField` | `null` | Data field dich de loc truong tuong thich tu dong |
| `description` | `tDataField("formula.selectDataFieldToInsert")` | Mo ta hien thi trong modal |
| `filterDataField` | `() => true` | Ham loc truong hien thi trong tree |
| `checkSelectableField` | `defaultFilterSelectableField` (dua tren `selectedDataField`) | Ham kiem tra truong co the chon |
