## Cách sử dụng component FormControlLabel trong ui-kit

Path: `ui-kit/src/components/FormControlLabel`

Props: `label` (ReactNode), `control` (ReactNode), `labelPlacement` ("top"|"left"|"right"|"bottom", default "right"), `labelAlign` ("left"|"right"|"center"), `disabled`, `disabledLabel` (default true), `usingHtmlFor` (default true), `className`, `classNameLabel`, `name`.

---

### Usage

```tsx
// Basic — label right of control (default)
<FormControlLabel
  label="Đồng ý điều khoản"
  control={<Checkbox checked={value} onChange={handleChange} />}
/>

// Label placement variants
<FormControlLabel label="Label" control={<Checkbox checked={value} onChange={handleChange} />} labelPlacement="left" />
<FormControlLabel label="Label" control={<Checkbox checked={value} onChange={handleChange} />} labelPlacement="top" />
<FormControlLabel label="Label" control={<Checkbox checked={value} onChange={handleChange} />} labelPlacement="bottom" />

// Label alignment (works with left/right placement — 140px fixed width)
<FormControlLabel label="Label" control={<Checkbox checked={value} onChange={handleChange} />} labelPlacement="left" labelAlign="right" />

// Disabled sync — disables both control and label styling
<FormControlLabel label="Tùy chọn" control={<Checkbox checked={value} onChange={handleChange} />} disabled />

// Keep label color when disabled
<FormControlLabel label="Tùy chọn" control={<Checkbox checked={value} onChange={handleChange} />} disabled disabledLabel={false} />

// Custom component not supporting htmlFor
<FormControlLabel label="Tùy chọn" control={<CustomToggle onToggle={handleToggle} />} usingHtmlFor={false} />

// Style label vs container
<FormControlLabel
  label="Tùy chọn"
  control={<Checkbox checked={value} onChange={handleChange} />}
  className={cx("mb-[1rem]")}
  classNameLabel={cx("text-typo-secondary prose-body1")}
/>

// Complex label content
<FormControlLabel
  label={<span>Đồng ý với <a href="/terms">điều khoản</a></span>}
  control={<Checkbox checked={value} onChange={handleChange} />}
/>
```

---

### DO / DON'T

```tsx
// ❌ Do NOT pass control via children — use control prop
<FormControlLabel label="Tùy chọn A">
  <Checkbox value="a" />
</FormControlLabel>

// ❌ Do NOT create layout manually
<div className={cx("flex items-center gap-[0.5rem]")}>
  <Checkbox checked={value} onChange={handleChange} />
  <span>Label</span>
</div>

// ❌ Do NOT override layout via className
<FormControlLabel label="Label" control={<Checkbox />} className={cx("!flex-row-reverse")} />

// ❌ Do NOT style label via container className selector
<FormControlLabel label="Label" control={<Checkbox />} className={cx("[&>label]:text-typo-secondary")} />

// ✅ Use classNameLabel for label, className for container
<FormControlLabel label="Label" control={<Checkbox />} classNameLabel={cx("prose-body2")} className={cx("mb-[1rem]")} />
```
