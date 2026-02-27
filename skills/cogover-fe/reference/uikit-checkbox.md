## Cách sử dụng component Checkbox và CheckboxGroup trong ui-kit

Paths:
- `ui-kit/src/components/Checkbox`
- `ui-kit/src/components/CheckboxGroup`
- `ui-kit/src/components/FormControlLabel`

**Checkbox** props: `checked`, `onChange`, `disabled`, `indeterminate` (default false), `size` ("large"|"medium"|"small", default "medium"), `value`, `className`. Omits `size|defaultChecked|value` from InputHTMLAttributes.

**CheckboxGroup** props: `value` (T[]), `onChange` ((e, value: T[]) => void), `row` (default false), `disabled`, `className`.

---

### Usage

```tsx
// Controlled Checkbox
<Checkbox checked={isAgree} onChange={(e) => setIsAgree(e.target.checked)} />

// Uncontrolled
<Checkbox onChange={(e) => handleChange(e)} />

// Indeterminate (select-all pattern)
<Checkbox
  checked={allChecked}
  indeterminate={!allChecked && someChecked}
  onChange={handleToggleAll}
/>

// Sizes
<Checkbox checked={value} onChange={handleChange} size="large" />
<Checkbox checked={value} onChange={handleChange} size="small" />

// CheckboxGroup — handles array toggle automatically
const [selected, setSelected] = useState<string[]>([]);
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" />
  <Checkbox value="b" />
</CheckboxGroup>

// With labels via FormControlLabel
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <FormControlLabel label="Lựa chọn A" control={<Checkbox value="a" />} />
  <FormControlLabel label="Lựa chọn B" control={<Checkbox value="b" />} />
</CheckboxGroup>

// Horizontal layout
<CheckboxGroup row value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" />
  <Checkbox value="b" />
</CheckboxGroup>

// Disable entire group vs specific items
<CheckboxGroup disabled value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" />
</CheckboxGroup>

<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" />
  <Checkbox value="b" disabled />
</CheckboxGroup>
```

---

### DO / DON'T

```tsx
// ❌ checked without onChange — frozen
<Checkbox checked={isAgree} />

// ❌ Do NOT use defaultChecked — omitted from type
<Checkbox defaultChecked={true} onChange={handleChange} />

// ❌ Do NOT manage array toggle manually
<div>
  <Checkbox value="a" checked={selected.includes("a")} onChange={(e) => {
    if (e.target.checked) setSelected([...selected, "a"]);
    else setSelected(selected.filter(v => v !== "a"));
  }} />
</div>

// ❌ Do NOT override size via className
<Checkbox className={cx("w-[2rem] h-[2rem]")} />

// ❌ Each checkbox must have unique value in group
<CheckboxGroup value={selected} onChange={(e, v) => setSelected(v)}>
  <Checkbox value="same" /><Checkbox value="same" />
</CheckboxGroup>

// ✅ CheckboxGroup handles all toggle logic
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="option1" />
  <Checkbox value="option2" />
</CheckboxGroup>
```
