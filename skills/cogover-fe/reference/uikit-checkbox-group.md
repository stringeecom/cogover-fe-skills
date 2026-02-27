## Cách sử dụng component CheckboxGroup trong ui-kit

Path: `ui-kit/src/components/CheckboxGroup`

Props: `value` (T[]), `onChange` ((e, value: T[]) => void), `row` (default false), `disabled`, `className`, `name`. Generic `CheckboxGroup<T>`.

Children: `Checkbox` directly or `FormControlLabel` with `control={<Checkbox />}`. Auto-handles add/remove from array on toggle.

---

### Usage

```tsx
// Basic — value must be array, onChange receives updated array
const [selected, setSelected] = useState<string[]>([]);

<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="option1" />
  <Checkbox value="option2" />
  <Checkbox value="option3" />
</CheckboxGroup>

// With labels
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <FormControlLabel label="Lựa chọn A" control={<Checkbox value="a" />} />
  <FormControlLabel label="Lựa chọn B" control={<Checkbox value="b" />} />
</CheckboxGroup>

// Horizontal layout
<CheckboxGroup row value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" />
  <Checkbox value="b" />
</CheckboxGroup>

// Disable entire group
<CheckboxGroup disabled value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" />
  <Checkbox value="b" />
</CheckboxGroup>

// Disable specific items — don't pass disabled on group
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value="a" />
  <Checkbox value="b" disabled />
  <Checkbox value="c" />
</CheckboxGroup>

// Numeric type — T inferred from value array
const [selected, setSelected] = useState<number[]>([]);
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)}>
  <Checkbox value={1} />
  <Checkbox value={2} />
</CheckboxGroup>

// Add className directly — CheckboxGroup renders flex div
<CheckboxGroup value={selected} onChange={(e, newValue) => setSelected(newValue)} className={cx("mt-[1rem]")}>
  <Checkbox value="a" />
</CheckboxGroup>
```

---

### DO / DON'T

```tsx
// ❌ Missing value — crashes on value.includes()
<CheckboxGroup onChange={(e, newValue) => setSelected(newValue)}>...</CheckboxGroup>

// ❌ Don't set checked per child — CheckboxGroup handles it
<CheckboxGroup value={selected} onChange={(e, v) => setSelected(v)}>
  <Checkbox value="a" checked={selected.includes("a")} />
</CheckboxGroup>

// ❌ Don't write array toggle manually
<Checkbox value="a" checked={selected.includes("a")} onChange={(e) => {
  if (e.target.checked) setSelected([...selected, "a"]);
  else setSelected(selected.filter(v => v !== "a"));
}} />

// ❌ Each checkbox needs unique value
<CheckboxGroup value={selected} onChange={(e, v) => setSelected(v)}>
  <Checkbox value="same" /><Checkbox value="same" />
</CheckboxGroup>

// ❌ Inconsistent types — value is string[] but checkbox value is number
const [selected, setSelected] = useState<string[]>([]);
<CheckboxGroup value={selected}><Checkbox value={1} /></CheckboxGroup>

// ❌ Don't wrap in extra div — CheckboxGroup already renders flex div
<div className={cx("flex flex-col gap-[0.75rem]")}>
  <CheckboxGroup value={selected} onChange={(e, v) => setSelected(v)}>...</CheckboxGroup>
</div>
```
