## Cách sử dụng InputNumber Component của ui-kit

Path: `ui-kit/src/components/InputNumber`

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `format` | `string` | `"###0"` | Display format string |
| `min` | `number` | `undefined` | Minimum value |
| `max` | `number` | `undefined` | Maximum value |
| `step` | `number` | `1` | Increment/decrement step |
| `value` | `number\|null` | `undefined` | Controlled value |
| `defaultValue` | `number\|null` | `undefined` | Uncontrolled initial value |
| `onChange` | `(value: number\|null) => void` | `undefined` | Change callback |
| `fillMissingZero` | `boolean` | `false` | Auto-fill decimal zeros |
| `numberOfIntegerDigit` | `number` | `15` | Max integer digits |
| `numberOfDecimalDigit` | `number` | `15` | Max decimal digits |
| `showIncreaseDecreaseButton` | `boolean` | `true` | Show +/- buttons |
| `roundRule` | `"round"\|"round_up"\|"round_down"\|null` | `undefined` | Rounding rule |
| `precision` | `number` | `2` | Decimal places when rounding |
| `allowTypeOutsideRange` | `boolean` | `true` | Allow typing outside min/max |

Inherits from `TextField`: `label`, `error`, `helperText`, `fullWidth`, `startAdornment`, `endAdornment`, `disabled`, `placeholder`, `className`, `inputClassName`, `readOnly`, `autoFocus`, and all `React.InputHTMLAttributes<HTMLInputElement>`.

**Format options:**
- Integer: `"###0"` (default), `"# ##0"` (space), `"#,##0"` (US comma), `"#.##0"` (DE dot)
- Decimal: append `.00` (dot) or `,00` (comma) — e.g. `"#,##0.00"`, `"#.##0,00"`

---

### Usage

```tsx
// Basic — onChange gets number|null directly (not event)
<InputNumber value={quantity} onChange={(value) => setQuantity(value)} />

// Currency with comma separator and 2 decimals
<InputNumber format="#,##0.00" value={price} onChange={setPrice} />

// Auto-fill decimal zeros (1.5 → 1.50 on blur)
<InputNumber format="#,##0.00" fillMissingZero value={price} onChange={setPrice} />

// Clamp to range on blur
<InputNumber min={0} max={100} allowTypeOutsideRange={false} value={amount} onChange={setAmount} />

// Rounding
<InputNumber format="#,##0.00" roundRule="round" precision={2} value={price} onChange={setPrice} />

// Hide +/- buttons, show custom endAdornment
<InputNumber showIncreaseDecreaseButton={false} endAdornment={<span>VND</span>} />

// Limit digit count
<InputNumber numberOfIntegerDigit={10} numberOfDecimalDigit={2} allowTypeOutsideRange={false} format="#,##0.00" value={amount} onChange={setAmount} />
```

---

### DO / DON'T

```tsx
// ❌ onChange is NOT an event handler
<InputNumber onChange={(e) => setQuantity(Number(e.target.value))} />

// ❌ Don't self-clamp — use allowTypeOutsideRange={false}
<InputNumber min={0} max={100} onChange={(value) => setAmount(Math.min(100, Math.max(0, value ?? 0)))} />

// ❌ Don't pass redundant defaults
<InputNumber format="###0" step={1} showIncreaseDecreaseButton={true} allowTypeOutsideRange={true} />

// ✅ Correct — only pass what differs from defaults
<InputNumber value={quantity} onChange={setQuantity} />
```
