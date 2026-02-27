## Cách sử dụng component RadioGroup trong ui-kit

Paths:
- `ui-kit/src/components/RadioGroup`
- `ui-kit/src/components/Radio`
- `ui-kit/src/components/FormControlLabel`

**RadioGroup** props: `value` (T|null), `onChange` ((e, value: T) => void), `row`, `disabled`, `className`, `name`. Generic `RadioGroupProps<T>`.

**Radio** props: `checked`, `onChange`, `disabled`, `size` ("large"|"medium"|"small", default "medium"), `value`, `className`. Omits `size|defaultChecked|value` from InputHTMLAttributes.

**FormControlLabel** props: `label`, `control`, `labelPlacement` ("top"|"left"|"right"|"bottom", default "right"), `labelAlign`, `disabled`, `disabledLabel` (default true), `usingHtmlFor` (default true), `className`, `classNameLabel`.

---

### Usage

```tsx
const [selected, setSelected] = useState<string | null>(null);

// Basic RadioGroup — handles checked state automatically
<RadioGroup value={selected} onChange={(e, val) => setSelected(val)}>
  <Radio value="a" />
  <Radio value="b" />
</RadioGroup>

// With labels via FormControlLabel
<RadioGroup value={selected} onChange={(e, val) => setSelected(val)}>
  <FormControlLabel label="Lựa chọn A" control={<Radio value="a" />} />
  <FormControlLabel label="Lựa chọn B" control={<Radio value="b" />} />
</RadioGroup>

// Horizontal layout
<RadioGroup row value={selected} onChange={(e, val) => setSelected(val)}>
  <FormControlLabel label="A" control={<Radio value="a" />} />
  <FormControlLabel label="B" control={<Radio value="b" />} />
</RadioGroup>

// Disable entire group
<RadioGroup disabled value={selected} onChange={(e, val) => setSelected(val)}>
  <Radio value="a" />
  <Radio value="b" />
</RadioGroup>

// Numeric values — use second param, not e.target.value (avoids string coercion)
<RadioGroup value={selected} onChange={(e, val) => setSelected(val)}>
  <Radio value={1} />
  <Radio value={2} />
</RadioGroup>

// Label placement on FormControlLabel
<FormControlLabel label="Label" control={<Radio value="a" />} labelPlacement="left" />
```

---

### DO / DON'T

```tsx
// ❌ Don't manage checked per Radio inside RadioGroup
<RadioGroup value={selected} onChange={(e, val) => setSelected(val)}>
  <Radio value="a" checked={selected === "a"} />  // redundant
  <Radio value="b" checked={selected === "b"} />  // redundant
</RadioGroup>

// ❌ Don't use e.target.value — loses type (number becomes string)
onChange={(e) => setSelected(e.target.value)}

// ❌ Don't override layout via className
<RadioGroup className={cx("!flex-row !items-center")} ...>

// ❌ Each Radio must have unique value
<RadioGroup ...><Radio value="same" /><Radio value="same" /></RadioGroup>

// ✅ Correct
<RadioGroup value={selected} onChange={(e, val) => setSelected(val)}>
  <Radio value="option1" />
  <Radio value="option2" />
</RadioGroup>
```
