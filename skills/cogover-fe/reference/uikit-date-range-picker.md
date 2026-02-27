## Cách sử dụng component DateRangePicker trong ui-kit

Path: `ui-kit/src/components/DateRangePicker`

Props: `value` ([Dayjs|null, Dayjs|null]), `onChange`, `format`, `fullWidth`, `error`, `helperText`, `color`, `showBorderOnBlur`, `locale`, `calendarProps`, `popperProps`, `renderValue`, `fromMin`, `fromMax`, `toMin`, `toMax`, `getFromRef`, `getToRef`, `dataTestId`, `onBlur`. Extends `React.InputHTMLAttributes<HTMLInputElement>`.

Defaults: `format="DD/MM/YYYY"`, `showBorderOnBlur=true`, `fullWidth=false`.

---

### Usage

```tsx
import dayjs, { Dayjs } from "dayjs";
import { useState } from "react";
import { DateRangePicker } from "ui-kit";

const [range, setRange] = useState<[Dayjs | null, Dayjs | null]>([null, null]);

// Basic date-only
<DateRangePicker value={range} onChange={setRange} format="DD/MM/YYYY" />

// DateTime mode — triggered automatically when format contains time tokens
<DateRangePicker value={range} onChange={setRange} format="DD/MM/YYYY HH:mm:ss" />

// Separate min/max for from and to
<DateRangePicker
  value={range}
  onChange={setRange}
  fromMin={dayjs().subtract(30, "day")}
  fromMax={dayjs()}
  toMin={dayjs()}
  toMax={dayjs().add(30, "day")}
/>

// Custom display
<DateRangePicker
  value={range}
  onChange={setRange}
  renderValue={(dateValue) => dateValue ? dateValue.format("dddd, DD/MM/YYYY") : ""}
/>

// Customize calendar via calendarProps
<DateRangePicker
  value={range}
  onChange={setRange}
  calendarProps={{ headerTextFormat: "MMMM YYYY", timeSelectProps: { format: "HH:mm" } }}
/>

// Error state
<DateRangePicker value={range} onChange={setRange} error={isError} helperText="Vui lòng chọn khoảng ngày" />

// Input refs
const [fromRef, setFromRef] = useState<HTMLInputElement | null>(null);
<DateRangePicker value={range} onChange={setRange} getFromRef={setFromRef} getToRef={setToRef} />
```

---

### DO / DON'T

```tsx
// ❌ Wrong value types
<DateRangePicker value={[new Date(), new Date()]} onChange={setRange} />
<DateRangePicker value={["2024-01-01", "2024-01-31"]} onChange={setRange} />

// ❌ Shared min/max — no such props, use fromMin/fromMax/toMin/toMax
<DateRangePicker value={range} onChange={setRange} min={dayjs()} max={dayjs()} />

// ❌ Self-validate swap — component auto-swaps from/to when from > to on blur
onChange={(newRange) => {
  if (newRange[0]?.isAfter(newRange[1])) setRange([newRange[1], newRange[0]]);
  else setRange(newRange);
}}

// ✅ Let component handle swap automatically
<DateRangePicker value={range} onChange={setRange} />
```
