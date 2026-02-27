## Cách sử dụng component TimePicker trong ui-kit

Path: `ui-kit/src/components/TimePicker`

Props: `value` (Dayjs|null), `onChange`, `format`, `min`, `max`, `renderValue`, `timeSelectProps`, `popperProps`, `helperText`, `hidePopperOnClear`, `showBorderOnBlur`, `className`. Extends `TextFieldProps` (except `value`, `onChange`, `max`, `min`).

Defaults: `format="HH:mm:ss"`, `showBorderOnBlur=true`, `hidePopperOnClear=false`, `timeSelectProps={}`.

---

### Usage

```tsx
import dayjs, { Dayjs } from "dayjs";
import { useState } from "react";
import { TimePicker } from "ui-kit";

const [time, setTime] = useState<Dayjs | null>(null);

// Basic — value must be Dayjs|null, not string or Date
<TimePicker value={time} onChange={setTime} />

// Custom format
<TimePicker value={time} onChange={setTime} format="HH:mm" />
<TimePicker value={time} onChange={setTime} format="hh:mm A" />

// min/max — restrict TimeSelect options
<TimePicker
  value={time}
  onChange={setTime}
  min={dayjs().hour(8).minute(0).second(0)}
  max={dayjs().hour(17).minute(0).second(0)}
/>

// Custom display — renderValue shows in input when not focused
<TimePicker
  value={time}
  onChange={setTime}
  renderValue={(value) => value ? value.format("HH giờ mm phút") : ""}
/>

// Error state
<TimePicker value={time} onChange={setTime} error={hasError} helperText={hasError ? "Vui lòng chọn giờ" : undefined} />

// Close popup when clearing
<TimePicker value={time} onChange={setTime} hidePopperOnClear />

// Customize internal TimeSelect
<TimePicker value={time} onChange={setTime} timeSelectProps={{ height: 300 }} />

// Customize Popper position
<TimePicker value={time} onChange={setTime} popperProps={{ placement: "top-start" }} />
```

---

### DO / DON'T

```tsx
// ❌ Wrong value types
<TimePicker value={new Date()} onChange={setTime} />
<TimePicker value="14:30:00" onChange={setTime} />
<TimePicker value={1705276800000} onChange={setTime} />

// ✅ Correct
<TimePicker value={dayjs("14:30:00", "HH:mm:ss")} onChange={setTime} />
<TimePicker value={null} onChange={setTime} />

// ❌ Self-format then pass string
<TimePicker value={dayjs().format("HH:mm")} onChange={setTime} />

// ❌ Self-validate in onChange — conflicts with internal logic
onChange={(newTime) => { if (newTime && newTime.isAfter(dayjs().hour(17))) return; setTime(newTime); }}

// ❌ Self-build clear button — built-in icon handles this
<div><TimePicker value={time} onChange={setTime} /><button onClick={() => setTime(null)}>×</button></div>

// ❌ Self-wrap Popper — creates nested popup
<Popper><TimePicker value={time} onChange={setTime} /></Popper>
```
