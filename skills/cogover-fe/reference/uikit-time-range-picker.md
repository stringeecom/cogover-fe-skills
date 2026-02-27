## Cách sử dụng component TimeRangePicker trong ui-kit

Path: `ui-kit/src/components/TimeRangePicker`

Props: `value` ([Dayjs|null, Dayjs|null]), `onChange`, `format`, `showBorderOnBlur`, `timeSelectProps`, `renderValue`, `fullWidth`, `helperText`, `color`, `error`, `className`, `popperProps`, `getFromRef`, `getToRef`, `fromMax`, `fromMin`, `toMax`, `toMin`, `onBlur`. Extends `React.InputHTMLAttributes<HTMLInputElement>` (except `value`, `onChange`, `onBlur`, `max`, `min`).

Defaults: `format="HH:mm:ss"`, `showBorderOnBlur=true`, `fullWidth=false`, `timeSelectProps={}`.

---

### Usage

```tsx
import dayjs, { Dayjs } from "dayjs";
import { useState } from "react";
import { TimeRangePicker } from "ui-kit";

// State must be tuple type
const [range, setRange] = useState<[Dayjs | null, Dayjs | null]>([null, null]);

// Basic — onChange returns full tuple
<TimeRangePicker value={range} onChange={setRange} />

// With format
<TimeRangePicker value={range} onChange={setRange} format="HH:mm" />

// Separate min/max for from and to inputs
<TimeRangePicker
  value={range}
  onChange={setRange}
  fromMin={dayjs("08:00:00", "HH:mm:ss")}
  fromMax={dayjs("12:00:00", "HH:mm:ss")}
  toMin={dayjs("13:00:00", "HH:mm:ss")}
  toMax={dayjs("17:00:00", "HH:mm:ss")}
/>

// Custom display via renderValue
<TimeRangePicker
  value={range}
  onChange={setRange}
  renderValue={(val) => val ? val.format("HH[h]mm[m]ss[s]") : ""}
/>

// Error state
<TimeRangePicker
  value={range}
  onChange={setRange}
  error={hasError}
  helperText={hasError ? "Thời gian không hợp lệ" : undefined}
/>

// Access input refs
const fromInputRef = useRef<HTMLInputElement | null>(null);
<TimeRangePicker
  value={range}
  onChange={setRange}
  getFromRef={(ref) => { fromInputRef.current = ref; }}
  getToRef={(ref) => { toInputRef.current = ref; }}
/>
```

---

### DO / DON'T

```tsx
// ❌ Wrong value types
<TimeRangePicker value={["08:00:00", "17:00:00"]} onChange={setRange} />
<TimeRangePicker value={[new Date(), new Date()]} onChange={setRange} />

// ❌ Wrong state type
const [range, setRange] = useState<Dayjs[]>([]); // must be tuple

// ❌ Shared min/max — no such props, use fromMin/fromMax/toMin/toMax
<TimeRangePicker value={range} onChange={setRange} min={...} max={...} />

// ❌ Custom clear button — icon is built-in
<div><TimeRangePicker value={range} onChange={setRange} /><button onClick={() => setRange([null, null])}>×</button></div>

// ✅ Programmatic clear
<TimeRangePicker value={range} onChange={setRange} />
<Button onClick={() => setRange([null, null])}>Reset</Button>
```
