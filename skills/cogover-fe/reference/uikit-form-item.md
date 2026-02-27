---
title: Cách sử dụng component FormItem trong ui-kit
impact: HIGH
impactDescription: Sử dụng FormItem sai dẫn đến layout label-component không nhất quán, helper text bị thiếu, và mất các chỉ báo required/readOnly
tags: form-item, form, label, layout, helper-text, ui-kit
---

## Cách sử dụng component FormItem trong ui-kit

Đường dẫn component: `ui-kit/src/components/FormItem`

**FormItem** props: `label` (ReactNode), `component` (ReactNode), `required`, `fullWidth`, `className`, `labelProps`, `noOutLine`, `helperText`, `subTitle`, `hint`, `rightIcon`, `rightEndComponent`, `previewComponent`, `helperTextTabIndex`, `labelPlacement`, `showLabel`, `labelAlign`, `readOnly`, `helpButtonProps`, `dataTestId`, `labelContainerClassName`.

Giá trị mặc định: `labelPlacement="top"`, `showLabel=true`, `labelAlign="left"`, `noOutLine=false`, `helperTextTabIndex=-1`.

Kiểu dữ liệu:
- `labelPlacement`: `"top" | "left" | "right" | "bottom"`
- `labelAlign`: `"left" | "right" | "center"`
- `readOnly`: `boolean | number` — khi truthy thì hiển thị icon khóa bên cạnh label

---

### RULE-FI-01: Sử dụng `FormItem` để tạo layout label + component — không tự tạo layout bằng div

`FormItem` tự động xử lý layout giữa label và component, bao gồm spacing, placement, alignment, required indicator, và readOnly icon. Không tự tạo layout thủ công.

**Sai:**

```tsx
// ❌ Tự tạo layout label + component — thiếu required indicator, readOnly icon, helper text
<div className={cx("flex flex-col")}>
  <label className={cx("text-typo-primary prose-body1")}>
    Tên khách hàng <span className={cx("text-error")}>*</span>
  </label>
  <div className={cx("mt-[0.5rem]")}>
    <Input value={name} onChange={handleChange} />
  </div>
</div>
```

**Đúng:**

```tsx
// ✅ FormItem tự xử lý layout, required indicator, spacing
<FormItem
  label="Tên khách hàng"
  component={<Input value={name} onChange={handleChange} />}
  required
/>
```

---

### RULE-FI-02: Truyền form control vào prop `component` — không truyền vào children

`FormItem` sử dụng prop `component` để nhận form control (Input, Select, DatePicker, ...). Component không hỗ trợ children.

**Sai:**

```tsx
// ❌ Truyền control qua children — không hoạt động
<FormItem label="Email">
  <Input value={email} onChange={handleChange} />
</FormItem>
```

**Đúng:**

```tsx
// ✅ Truyền qua prop component
<FormItem
  label="Email"
  component={<Input value={email} onChange={handleChange} />}
/>
```

---

### RULE-FI-03: Sử dụng `labelPlacement` để đổi vị trí label — không override flex direction qua className

`FormItem` hỗ trợ `labelPlacement`: `"top"` (mặc định), `"left"`, `"right"`, `"bottom"`. Component tự xử lý flex direction tương ứng. Khi `labelPlacement` là `"left"` hoặc `"right"`, label container có width cố định `140px` và component chiếm phần còn lại.

**Sai:**

```tsx
// ❌ Override flex direction qua className
<FormItem
  label="Tên"
  component={<Input value={name} onChange={handleChange} />}
  className={cx("!flex-row")}
/>
```

**Đúng:**

```tsx
// ✅ Label bên trái (mặc định width 140px)
<FormItem
  label="Tên"
  component={<Input value={name} onChange={handleChange} />}
  labelPlacement="left"
/>

// ✅ Label phía dưới
<FormItem
  label="Tên"
  component={<Input value={name} onChange={handleChange} />}
  labelPlacement="bottom"
/>
```

---

### RULE-FI-04: Sử dụng `labelAlign` để căn chỉnh nội dung label — không override alignment qua className hoặc labelProps

`labelAlign` căn chỉnh nội dung label: `"left"` (mặc định) dùng `justify-start`, `"right"` dùng `justify-end`, `"center"` dùng `justify-center`.

**Sai:**

```tsx
// ❌ Override alignment qua labelProps
<FormItem
  label="Tên"
  component={<Input value={name} onChange={handleChange} />}
  labelProps={{ className: cx("!justify-end") }}
/>
```

**Đúng:**

```tsx
// ✅ Căn phải label
<FormItem
  label="Tên"
  component={<Input value={name} onChange={handleChange} />}
  labelPlacement="left"
  labelAlign="right"
/>
```

---

### RULE-FI-05: Sử dụng prop `required` để hiển thị dấu sao đỏ — không tự thêm dấu sao vào label

Khi `required={true}`, `FormItem` tự động thêm dấu `*` màu `text-error` sau label. Không tự thêm dấu sao vào nội dung label.

**Sai:**

```tsx
// ❌ Tự thêm dấu sao vào label — bị trùng lặp nếu dùng chung với required
<FormItem
  label={<span>Tên khách hàng <span className={cx("text-error")}>*</span></span>}
  component={<Input value={name} onChange={handleChange} />}
/>
```

**Đúng:**

```tsx
// ✅ required tự thêm dấu sao đỏ
<FormItem
  label="Tên khách hàng"
  component={<Input value={name} onChange={handleChange} />}
  required
/>
```

---

### RULE-FI-06: Sử dụng prop `helperText` để hiển thị tooltip hướng dẫn — không tự tạo HelpButton

Khi truyền `helperText`, `FormItem` tự động render `HelpButton` bên cạnh label với popup tooltip chứa nội dung hướng dẫn. Có thể tùy chỉnh thêm qua `helpButtonProps` và `helperTextTabIndex`.

**Sai:**

```tsx
// ❌ Tự tạo HelpButton bên cạnh label
<FormItem
  label={
    <span>
      Mã số thuế
      <HelpButton helpText="Nhập mã số thuế 10 hoặc 13 số" />
    </span>
  }
  component={<Input value={taxCode} onChange={handleChange} />}
/>
```

**Đúng:**

```tsx
// ✅ Dùng prop helperText — FormItem tự tạo HelpButton
<FormItem
  label="Mã số thuế"
  component={<Input value={taxCode} onChange={handleChange} />}
  helperText="Nhập mã số thuế 10 hoặc 13 số"
/>

// ✅ Tùy chỉnh thêm HelpButton qua helpButtonProps
<FormItem
  label="Mã số thuế"
  component={<Input value={taxCode} onChange={handleChange} />}
  helperText="Nhập mã số thuế 10 hoặc 13 số"
  helpButtonProps={{ renderHTML: true }}
/>
```

---

### RULE-FI-07: Sử dụng prop `readOnly` để hiển thị icon khóa — không tự thêm icon vào label

Khi `readOnly` truthy, `FormItem` tự động hiển thị icon khóa (`faLockKeyhole`) bên cạnh label với màu `text-gray-50`. Không tự thêm icon khóa vào label.

**Sai:**

```tsx
// ❌ Tự thêm icon khóa vào label
<FormItem
  label={
    <span>
      Mã hợp đồng <FontAwesomeIcon icon={faLockKeyhole} className={cx("text-gray-50")} />
    </span>
  }
  component={<Input value={contractCode} readOnly />}
/>
```

**Đúng:**

```tsx
// ✅ readOnly tự thêm icon khóa
<FormItem
  label="Mã hợp đồng"
  component={<Input value={contractCode} readOnly />}
  readOnly
/>
```

---

### RULE-FI-08: Sử dụng `rightIcon` và `rightEndComponent` đúng mục đích

- `rightIcon`: Hiển thị icon nhỏ ngay sau text label (sau dấu required, trước helperText).
- `rightEndComponent`: Hiển thị component ở cuối hàng label (bên phải, cùng hàng với label).

**Sai:**

```tsx
// ❌ Nhét cả icon và button vào label
<FormItem
  label={
    <span className={cx("flex items-center justify-between w-full")}>
      <span>Ghi chú <InfoIcon /></span>
      <Button size="small">Thêm</Button>
    </span>
  }
  component={<TextArea value={note} onChange={handleChange} />}
/>
```

**Đúng:**

```tsx
// ✅ Dùng rightIcon cho icon, rightEndComponent cho component cuối hàng
<FormItem
  label="Ghi chú"
  component={<TextArea value={note} onChange={handleChange} />}
  rightIcon={<InfoIcon />}
  rightEndComponent={<Button size="small">Thêm</Button>}
/>
```

---

### RULE-FI-09: Sử dụng `hint` và `subTitle` để hiển thị thông tin phụ dưới label — không tự tạo thêm text

- `hint`: Hiển thị ngay dưới label với style `prose-body2 text-typo-primary`.
- `subTitle`: Hiển thị dưới hint với Typography variant `body2` và `mt-[0.5rem]`.

**Sai:**

```tsx
// ❌ Tự thêm text hướng dẫn dưới label
<div>
  <FormItem
    label="Số điện thoại"
    component={<Input value={phone} onChange={handleChange} />}
  />
  <span className={cx("prose-body2 text-typo-primary")}>Nhập số điện thoại 10 chữ số</span>
</div>
```

**Đúng:**

```tsx
// ✅ Dùng hint cho mô tả ngắn dưới label
<FormItem
  label="Số điện thoại"
  component={<Input value={phone} onChange={handleChange} />}
  hint="Nhập số điện thoại 10 chữ số"
/>

// ✅ Dùng subTitle cho nội dung phụ dưới hint
<FormItem
  label="Số điện thoại"
  component={<Input value={phone} onChange={handleChange} />}
  hint="Nhập số điện thoại 10 chữ số"
  subTitle="Hỗ trợ đầu số: 09x, 03x, 07x, 08x"
/>
```

---

### RULE-FI-10: Sử dụng `showLabel={false}` để ẩn label nhưng vẫn giữ hint/subTitle — không truyền label rỗng

Khi cần ẩn label nhưng vẫn giữ các thông tin phụ (hint, subTitle), sử dụng `showLabel={false}`. Nếu không truyền `label` hoặc truyền label rỗng, toàn bộ khối label (bao gồm hint, subTitle) sẽ không được render.

**Sai:**

```tsx
// ❌ Truyền label rỗng — hint và subTitle cũng bị ẩn
<FormItem
  component={<Input value={name} onChange={handleChange} />}
  hint="Gợi ý quan trọng"
/>
```

**Đúng:**

```tsx
// ✅ Ẩn label nhưng vẫn hiển thị hint
<FormItem
  label="Tên"
  component={<Input value={name} onChange={handleChange} />}
  showLabel={false}
  hint="Gợi ý quan trọng"
/>
```

---

### RULE-FI-11: Sử dụng `labelContainerClassName` để style container của label — sử dụng `labelProps` để style thẻ label

- `labelContainerClassName`: Áp dụng cho div bao ngoài label, hint, subTitle.
- `labelProps`: Áp dụng cho thẻ `<label>` (Typography component) bên trong.

**Sai:**

```tsx
// ❌ Dùng className để style label container
<FormItem
  label="Tên"
  component={<Input value={name} onChange={handleChange} />}
  className={cx("[&>.form-item-label]:w-[200px]")}
/>
```

**Đúng:**

```tsx
// ✅ Dùng labelContainerClassName cho container
<FormItem
  label="Tên"
  component={<Input value={name} onChange={handleChange} />}
  labelContainerClassName={cx("w-[200px]")}
/>

// ✅ Dùng labelProps cho thẻ label
<FormItem
  label="Tên"
  component={<Input value={name} onChange={handleChange} />}
  labelProps={{ className: cx("prose-body2") }}
/>
```

---

### RULE-FI-12: Sử dụng `previewComponent` để hiển thị nội dung xem trước dưới component

`previewComponent` render một div dưới phần component chính. Dùng để hiển thị preview, ảnh thumbnail, hoặc nội dung bổ sung.

**Sai:**

```tsx
// ❌ Tự wrap thêm div preview bên ngoài FormItem
<div>
  <FormItem
    label="Ảnh đại diện"
    component={<FileUpload onChange={handleUpload} />}
  />
  <div>
    <img src={previewUrl} alt="preview" />
  </div>
</div>
```

**Đúng:**

```tsx
// ✅ Dùng previewComponent
<FormItem
  label="Ảnh đại diện"
  component={<FileUpload onChange={handleUpload} />}
  previewComponent={<img src={previewUrl} alt="preview" />}
/>
```
