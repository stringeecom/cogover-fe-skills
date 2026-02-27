## Cách sử dụng component Radio trong ui-kit

Path: `ui-kit/src/components/Radio`

Props: `checked`, `onChange`, `disabled`, `size` ("large"|"medium"|"small", default "medium"), `value`, `className`. Omits `size|defaultChecked|value` from InputHTMLAttributes.

Sizes: `"large"` (24px), `"medium"` (20px, default), `"small"` (16px).

Note: type is always `"radio"` internally — do not pass `type` prop. `defaultChecked` is omitted from type.

For grouped radios, use `RadioGroup`. For labels, use `FormControlLabel`. See `uikit-radio-group.md` for group usage.

---

### Usage

```tsx
// Controlled
<Radio checked={isSelected} onChange={(e) => setIsSelected(e.target.checked)} />

// Uncontrolled
<Radio onChange={(e) => handleChange(e)} />

// Sizes
<Radio checked={value} onChange={handleChange} size="large" />
<Radio checked={value} onChange={handleChange} size="small" />

// With value (required when used inside RadioGroup)
<Radio value="option1" />

// With FormControlLabel for label
<FormControlLabel
  label="Lựa chọn A"
  control={<Radio checked={value} onChange={handleChange} />}
/>

// Label placement
<FormControlLabel
  label="Label"
  control={<Radio checked={value} onChange={handleChange} />}
  labelPlacement="left"
/>

// value supports number | string | boolean
<Radio value={1} checked={selected === 1} onChange={handleChange} />
<Radio value="option1" checked={selected === "option1"} onChange={handleChange} />
<Radio value={true} checked={selected === true} onChange={handleChange} />
```

---

### DO / DON'T

```tsx
// ❌ checked without onChange — radio frozen
<Radio checked={isSelected} />

// ❌ defaultChecked omitted from type
<Radio defaultChecked={true} onChange={handleChange} />

// ❌ type prop redundant — always "radio"
<Radio type="radio" checked={value} onChange={handleChange} />

// ❌ Override size via className — breaks border-radius and icon scaling
<Radio className={cx("w-[2rem] h-[2rem]")} checked={value} onChange={handleChange} />

// ❌ Don't manually manage multiple radios — use RadioGroup
<div>
  <Radio value="a" checked={selected === "a"} onChange={() => setSelected("a")} />
  <Radio value="b" checked={selected === "b"} onChange={() => setSelected("b")} />
</div>

// ✅ Use RadioGroup for groups
<RadioGroup value={selected} onChange={(e, val) => setSelected(val)}>
  <Radio value="a" />
  <Radio value="b" />
</RadioGroup>
```
