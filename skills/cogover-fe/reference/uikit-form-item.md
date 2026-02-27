## Cách sử dụng component FormItem trong ui-kit

Path: `ui-kit/src/components/FormItem`

Props: `label`, `component`, `required`, `fullWidth`, `className`, `labelProps`, `noOutLine`, `helperText`, `subTitle`, `hint`, `rightIcon`, `rightEndComponent`, `previewComponent`, `helperTextTabIndex`, `labelPlacement`, `showLabel`, `labelAlign`, `readOnly`, `helpButtonProps`, `dataTestId`, `labelContainerClassName`.

Defaults: `labelPlacement="top"`, `showLabel=true`, `labelAlign="left"`, `noOutLine=false`, `helperTextTabIndex=-1`.

Types: `labelPlacement`: `"top"|"left"|"right"|"bottom"` | `labelAlign`: `"left"|"right"|"center"` | `readOnly`: `boolean|number`

---

### Usage

```tsx
// ✅ Basic usage — component prop, not children
<FormItem
  label="Tên khách hàng"
  component={<Input value={name} onChange={handleChange} />}
  required
/>

// ✅ With helper text tooltip
<FormItem
  label="Mã số thuế"
  component={<Input value={taxCode} onChange={handleChange} />}
  helperText="Nhập mã số thuế 10 hoặc 13 số"
/>

// ✅ Left label placement
<FormItem
  label="Tên"
  component={<Input value={name} onChange={handleChange} />}
  labelPlacement="left"
  labelAlign="right"
/>

// ✅ With hint/subTitle below label
<FormItem
  label="Số điện thoại"
  component={<Input value={phone} onChange={handleChange} />}
  hint="Nhập số điện thoại 10 chữ số"
  subTitle="Hỗ trợ đầu số: 09x, 03x, 07x, 08x"
/>

// ✅ readOnly shows lock icon automatically
<FormItem
  label="Mã hợp đồng"
  component={<Input value={contractCode} readOnly />}
  readOnly
/>

// ✅ rightIcon + rightEndComponent
<FormItem
  label="Ghi chú"
  component={<TextArea value={note} onChange={handleChange} />}
  rightIcon={<InfoIcon />}
  rightEndComponent={<Button size="small">Thêm</Button>}
/>

// ✅ Hide label but show hint
<FormItem
  label="Tên"
  component={<Input value={name} onChange={handleChange} />}
  showLabel={false}
  hint="Gợi ý quan trọng"
/>

// ✅ Style label container vs label element
<FormItem
  label="Tên"
  component={<Input value={name} onChange={handleChange} />}
  labelContainerClassName={cx("w-[200px]")}
  labelProps={{ className: cx("prose-body2") }}
/>
```

---

### DO / DON'T

```tsx
// ❌ Do NOT pass control via children
<FormItem label="Email">
  <Input value={email} onChange={handleChange} />
</FormItem>

// ❌ Do NOT add asterisk manually to label
<FormItem
  label={<span>Tên <span className={cx("text-error")}>*</span></span>}
  component={<Input />}
/>

// ❌ Do NOT create layout manually
<div className={cx("flex flex-col")}>
  <label>Tên</label>
  <Input value={name} onChange={handleChange} />
</div>

// ✅ Use component prop + required for asterisk
<FormItem
  label="Tên"
  component={<Input value={name} onChange={handleChange} />}
  required
/>
```
