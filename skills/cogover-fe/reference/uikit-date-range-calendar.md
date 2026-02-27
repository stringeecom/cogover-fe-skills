---
title: Cách sử dụng component DateRangeCalendar trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến khoảng ngày highlight sai, trạng thái focus from/to không đúng, min/max không hoạt động, dateTime mode conflict
tags: date-range-calendar, date-range, calendar, range, dayjs, datetime, time-select, ui-kit
---

## Cách sử dụng component DateRangeCalendar trong ui-kit

Đường dẫn component: `ui-kit/src/components/DateRangeCalendar`

Props chính: `value` ([Dayjs | null, Dayjs | null]), `onChange`, `onChangeTime`, `headerTextFormat`, `disabledKeyboard`, `max`, `min`, `timeSelectProps`, `focusedValueIndex`, `dateTime`. Kế thừa các props từ `React.HTMLAttributes<HTMLDivElement>` (ngoại trừ `onChange`).

Giá trị mặc định: `headerTextFormat="MM/YYYY"`, `disabledKeyboard=false`, `dateTime=false`, `focusedValueIndex=0`.

---

### RULE-DATE-RANGE-CAL-01: `value` là tuple `[Dayjs | null, Dayjs | null]` — không truyền Date, string, hay mảng sai kiểu

DateRangeCalendar sử dụng `dayjs` library. `value` phải là tuple gồm 2 phần tử, mỗi phần tử là `Dayjs` instance hoặc `null`. Phần tử đầu tiên là ngày bắt đầu (from), phần tử thứ hai là ngày kết thúc (to).

**Sai:**

```tsx
// ❌ Truyền mảng Date
<DateRangeCalendar value={[new Date(), new Date()]} onChange={handleChange} />

// ❌ Truyền string
<DateRangeCalendar value={["2024-01-01", "2024-01-31"]} onChange={handleChange} />

// ❌ Truyền mảng rỗng
<DateRangeCalendar value={[]} onChange={handleChange} />

// ❌ Truyền một phần tử
<DateRangeCalendar value={[dayjs()]} onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅ Cả hai giá trị đều là Dayjs
<DateRangeCalendar value={[dayjs(), dayjs().add(7, "day")]} onChange={handleChange} />

// ✅ Chưa chọn giá trị nào
<DateRangeCalendar value={[null, null]} onChange={handleChange} />

// ✅ Chỉ chọn ngày bắt đầu
<DateRangeCalendar value={[dayjs(), null]} onChange={handleChange} />

// ✅ Chỉ chọn ngày kết thúc
<DateRangeCalendar value={[null, dayjs().add(7, "day")]} onChange={handleChange} />
```

---

### RULE-DATE-RANGE-CAL-02: `onChange` trả về một `Dayjs` duy nhất — tự quản lý logic from/to bên ngoài

`onChange` callback nhận một `Dayjs` đại diện cho ngày vừa được chọn. Component không tự quản lý logic gán vào from hay to. Phải tự xử lý logic này bên ngoài dựa trên trạng thái hiện tại.

**Sai:**

```tsx
// ❌ Mong đợi onChange trả về tuple [Dayjs, Dayjs]
<DateRangeCalendar
  value={value}
  onChange={(newRange) => {
    setValue(newRange); // newRange là Dayjs, không phải tuple
  }}
/>
```

**Đúng:**

```tsx
// ✅ Tự quản lý logic gán from/to
const [value, setValue] = useState<[Dayjs | null, Dayjs | null]>([null, null]);

<DateRangeCalendar
  value={value}
  onChange={(date) => {
    if (!value[0]) {
      setValue([date, null]);
    } else if (!value[1]) {
      setValue([value[0], date]);
    } else {
      setValue([date, null]);
    }
  }}
/>
```

---

### RULE-DATE-RANGE-CAL-03: Sử dụng `focusedValueIndex` để điều khiển focus vào from (0) hoặc to (1) — không tự build logic focus

`focusedValueIndex` nhận giá trị `0` hoặc `1` để xác định input nào đang được focus. Khi `dateTime=true`, giá trị này quyết định TimeSelect hiển thị giờ của from hay to. Calendar cũng tự cuộn đến tháng của giá trị tương ứng.

**Sai:**

```tsx
// ❌ Không truyền focusedValueIndex khi cần switch giữa from/to
<DateRangeCalendar
  value={value}
  dateTime
  onChange={handleChange}
/>
```

**Đúng:**

```tsx
// ✅ Chuyển focus giữa from và to
const [focusedIndex, setFocusedIndex] = useState<0 | 1>(0);

<DateRangeCalendar
  value={value}
  dateTime
  focusedValueIndex={focusedIndex}
  onChange={(date) => {
    if (focusedIndex === 0) {
      setValue([date, value[1]]);
      setFocusedIndex(1);
    } else {
      setValue([value[0], date]);
      setFocusedIndex(0);
    }
  }}
/>
```

---

### RULE-DATE-RANGE-CAL-04: Bật `dateTime` để hiển thị TimeSelect — không tự build time picker bên ngoài

Khi `dateTime=true`, component hiển thị TimeSelect bên cạnh calendar. Ngày được chọn sẽ tự gán giờ/phút/giây từ giá trị hiện tại hoặc giờ hiện tại. Dùng `timeSelectProps` để tuỳ chỉnh TimeSelect và `onChangeTime` để lắng nghe thay đổi giờ.

**Sai:**

```tsx
// ❌ Tự build time picker bên ngoài
<div className={cx("flex gap-[0.5rem]")}>
  <DateRangeCalendar value={value} onChange={handleChange} />
  <TimeSelect value={value[0]} onChange={handleTimeChange} />
</div>
```

**Đúng:**

```tsx
// ✅ Chỉ chọn ngày (mặc định)
<DateRangeCalendar value={value} onChange={handleChange} />

// ✅ Chọn ngày + giờ — TimeSelect tự hiện
<DateRangeCalendar
  value={value}
  dateTime
  focusedValueIndex={focusedIndex}
  onChange={handleChange}
  onChangeTime={(date) => {
    // Xử lý khi thay đổi giờ
  }}
/>

// ✅ Tuỳ chỉnh format giờ qua timeSelectProps
<DateRangeCalendar
  value={value}
  dateTime
  focusedValueIndex={focusedIndex}
  onChange={handleChange}
  timeSelectProps={{ format: "HH:mm" }}
/>
```

---

### RULE-DATE-RANGE-CAL-05: Sử dụng `min`/`max` là `Dayjs` để giới hạn khoảng ngày — không tự validate bên ngoài

`min` và `max` nhận `Dayjs` instance. Calendar tự disable các ngày ngoài khoảng, keyboard navigation tự dừng tại biên, và year options tự giới hạn theo min/max year.

**Sai:**

```tsx
// ❌ Tự validate trong onChange
<DateRangeCalendar
  value={value}
  onChange={(date) => {
    if (date.isAfter(dayjs())) return;
    handleChange(date);
  }}
/>

// ❌ Truyền Date hoặc string cho min/max
<DateRangeCalendar
  value={value}
  onChange={handleChange}
  min={new Date("2024-01-01")}
  max="2024-12-31"
/>
```

**Đúng:**

```tsx
// ✅ Giới hạn khoảng ngày bằng Dayjs
<DateRangeCalendar
  value={value}
  onChange={handleChange}
  min={dayjs().subtract(30, "day")}
  max={dayjs()}
/>

// ✅ Giới hạn theo năm
<DateRangeCalendar
  value={value}
  onChange={handleChange}
  min={dayjs("2024-01-01")}
  max={dayjs("2024-12-31")}
/>
```

---

### RULE-DATE-RANGE-CAL-06: Sử dụng `disabledKeyboard` để tắt điều hướng bàn phím khi dùng trong Popper/Popup

Khi sử dụng DateRangeCalendar bên trong Popper hoặc popup (như DateRangePicker), nên bật `disabledKeyboard=true` để tránh xung đột keyboard events giữa calendar và input bên ngoài.

**Sai:**

```tsx
// ❌ Không tắt keyboard khi dùng trong Popper — gây xung đột phím Enter/Arrow
<Popper render={() => <DateRangeCalendar value={value} onChange={handleChange} />}>
  {(params) => <input {...params} />}
</Popper>
```

**Đúng:**

```tsx
// ✅ Tắt keyboard khi dùng trong Popper
<Popper
  render={() => (
    <DateRangeCalendar value={value} onChange={handleChange} disabledKeyboard />
  )}
>
  {(params) => <input {...params} />}
</Popper>

// ✅ Bật keyboard khi dùng standalone
<DateRangeCalendar value={value} onChange={handleChange} />
```

---

### RULE-DATE-RANGE-CAL-07: Sử dụng `headerTextFormat` để tuỳ chỉnh hiển thị tháng/năm ở header — không tự build header

`headerTextFormat` nhận format string của dayjs để hiển thị tháng/năm. Mặc định là `"MM/YYYY"`. Header đã tích hợp sẵn nút prev/next month và chọn năm.

**Sai:**

```tsx
// ❌ Tự build header bên ngoài
<div>
  <div className={cx("flex justify-between")}>
    <button onClick={goToPrevMonth}>←</button>
    <span>{currentMonth.format("MMMM YYYY")}</span>
    <button onClick={goToNextMonth}>→</button>
  </div>
  <DateRangeCalendar value={value} onChange={handleChange} />
</div>
```

**Đúng:**

```tsx
// ✅ Mặc định MM/YYYY
<DateRangeCalendar value={value} onChange={handleChange} />

// ✅ Tuỳ chỉnh format header
<DateRangeCalendar value={value} onChange={handleChange} headerTextFormat="MMMM YYYY" />
```

---

### RULE-DATE-RANGE-CAL-08: Highlight khoảng ngày giữa from và to được xử lý tự động — không cần tuỳ chỉnh

Component tự highlight các ngày nằm giữa `value[0]` (from) và `value[1]` (to) bằng background `bg-primary-light-90`. Ngày from và to được highlight bằng `bg-primary-main`. Không cần tự thêm style hoặc logic highlight.

**Sai:**

```tsx
// ❌ Tự thêm logic highlight bên ngoài
<DateRangeCalendar
  value={value}
  onChange={handleChange}
  className={cx({
    "highlight-range": value[0] && value[1],
  })}
/>
```

**Đúng:**

```tsx
// ✅ Highlight tự động khi cả from và to có giá trị
const [value, setValue] = useState<[Dayjs | null, Dayjs | null]>([
  dayjs(),
  dayjs().add(7, "day"),
]);

<DateRangeCalendar value={value} onChange={handleChange} />
```

---

### Ví dụ hoàn chỉnh: DateRangeCalendar standalone với chọn khoảng ngày

```tsx
import dayjs, { Dayjs } from "dayjs";
import { useState } from "react";
import { DateRangeCalendar } from "ui-kit";
import cx from "ui-kit/utils/cx";

function DateRangeExample() {
  const [value, setValue] = useState<[Dayjs | null, Dayjs | null]>([dayjs(), dayjs().add(1, "day")]);

  return (
    <div className={cx("shadow-primary", "border-divider-primary", "rounded")}>
      <DateRangeCalendar
        value={value}
        onChange={(date) => {
          if (!value[0] || value[1]) {
            return setValue([date, null]);
          }

          if (value[0]) {
            return setValue([value[0], date]);
          }
        }}
      />
    </div>
  );
}
```

### Ví dụ hoàn chỉnh: DateRangeCalendar với dateTime mode

```tsx
import dayjs, { Dayjs } from "dayjs";
import { useState } from "react";
import { DateRangeCalendar } from "ui-kit";
import cx from "ui-kit/utils/cx";

function DateTimeRangeExample() {
  const [value, setValue] = useState<[Dayjs | null, Dayjs | null]>([null, null]);
  const [focusedIndex, setFocusedIndex] = useState<0 | 1>(0);

  return (
    <div className={cx("shadow-primary", "border-divider-primary", "rounded")}>
      <DateRangeCalendar
        value={value}
        dateTime
        focusedValueIndex={focusedIndex}
        timeSelectProps={{ format: "HH:mm" }}
        min={dayjs().subtract(30, "day")}
        max={dayjs().add(30, "day")}
        onChange={(date) => {
          if (focusedIndex === 0) {
            setValue([date, value[1]]);
            setFocusedIndex(1);
          } else {
            setValue([value[0], date]);
            setFocusedIndex(0);
          }
        }}
        onChangeTime={(date) => {
          if (focusedIndex === 0) {
            setValue([date, value[1]]);
            setFocusedIndex(1);
          } else {
            setValue([value[0], date]);
            setFocusedIndex(0);
          }
        }}
      />
    </div>
  );
}
```
