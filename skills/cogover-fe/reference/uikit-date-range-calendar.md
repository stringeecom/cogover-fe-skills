## Cách sử dụng component DateRangeCalendar trong ui-kit

Path: `ui-kit/src/components/DateRangeCalendar`

Props: `value` ([Dayjs|null, Dayjs|null]), `onChange`, `onChangeTime`, `headerTextFormat`, `disabledKeyboard`, `max`, `min`, `timeSelectProps`, `focusedValueIndex`, `dateTime`. Extends `React.HTMLAttributes<HTMLDivElement>` (except `onChange`).

Defaults: `headerTextFormat="MM/YYYY"`, `disabledKeyboard=false`, `dateTime=false`, `focusedValueIndex=0`.

---

### Usage

```tsx
import dayjs, { Dayjs } from "dayjs";
import { useState } from "react";
import { DateRangeCalendar } from "ui-kit";

// Basic date range selection
function BasicExample() {
  const [value, setValue] = useState<[Dayjs | null, Dayjs | null]>([dayjs(), dayjs().add(1, "day")]);

  return (
    <DateRangeCalendar
      value={value}
      onChange={(date) => {
        // onChange returns single Dayjs — manage from/to logic externally
        if (!value[0] || value[1]) return setValue([date, null]);
        if (value[0]) return setValue([value[0], date]);
      }}
      min={dayjs().subtract(30, "day")}
      max={dayjs().add(30, "day")}
    />
  );
}

// DateTime mode with focusedValueIndex
function DateTimeExample() {
  const [value, setValue] = useState<[Dayjs | null, Dayjs | null]>([null, null]);
  const [focusedIndex, setFocusedIndex] = useState<0 | 1>(0);

  return (
    <DateRangeCalendar
      value={value}
      dateTime
      focusedValueIndex={focusedIndex}
      timeSelectProps={{ format: "HH:mm" }}
      onChange={(date) => {
        if (focusedIndex === 0) { setValue([date, value[1]]); setFocusedIndex(1); }
        else { setValue([value[0], date]); setFocusedIndex(0); }
      }}
      onChangeTime={(date) => {
        if (focusedIndex === 0) setValue([date, value[1]]);
        else setValue([value[0], date]);
      }}
    />
  );
}

// In Popper — disable keyboard to avoid conflicts
<Popper render={() => (
  <DateRangeCalendar value={value} onChange={handleChange} disabledKeyboard />
)}>
  {(params) => <input {...params} />}
</Popper>
```

---

### DO / DON'T

```tsx
// ❌ Wrong value types
<DateRangeCalendar value={[new Date(), new Date()]} onChange={handleChange} />
<DateRangeCalendar value={["2024-01-01", "2024-01-31"]} onChange={handleChange} />
<DateRangeCalendar value={[dayjs()]} onChange={handleChange} /> // single element

// ❌ Expecting onChange to return tuple — it returns single Dayjs
onChange={(newRange) => setValue(newRange)} // newRange is Dayjs, not tuple

// ❌ Self-build time picker alongside calendar
<div><DateRangeCalendar value={value} onChange={handleChange} />
<TimeSelect value={value[0]} onChange={handleTimeChange} /></div>

// ✅ Use dateTime prop instead
<DateRangeCalendar value={value} dateTime focusedValueIndex={focusedIndex} onChange={handleChange} />
```
