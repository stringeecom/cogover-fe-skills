---
title: Cách sử dụng component DateRangePicker trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến khoảng ngày hiển thị sai format, calendar không mở, min/max từ/đến không hoạt động, from/to bị đảo ngược
tags: date-range-picker, date-range, date, calendar, dayjs, datetime, range, ui-kit
---

## Cách sử dụng component DateRangePicker trong ui-kit

Đường dẫn component: `ui-kit/src/components/DateRangePicker`

Props chính: `value` ([Dayjs | null, Dayjs | null]), `onChange`, `format`, `fullWidth`, `error`, `helperText`, `color`, `showBorderOnBlur`, `locale`, `calendarProps`, `popperProps`, `renderValue`, `fromMin`, `fromMax`, `toMin`, `toMax`, `getFromRef`, `getToRef`, `dataTestId`, `onBlur`. Kế thừa các props từ `React.InputHTMLAttributes<HTMLInputElement>`.

Giá trị mặc định: `format="DD/MM/YYYY"`, `showBorderOnBlur=true`, `fullWidth=false`.

---

### RULE-DATE-RANGE-PICKER-01: `value` là `[Dayjs | null, Dayjs | null]` — không truyền Date, string, hay timestamp

DateRangePicker sử dụng `dayjs` library. `value` phải là một tuple gồm 2 phần tử, mỗi phần tử là instance `Dayjs` hoặc `null`. Phần tử đầu là ngày bắt đầu (from), phần tử sau là ngày kết thúc (to).

**Sai:**

```tsx
// ❌ Native Date objects
<DateRangePicker value={[new Date(), new Date()]} onChange={setRange} />

// ❌ ISO strings
<DateRangePicker value={["2024-01-01", "2024-01-31"]} onChange={setRange} />

// ❌ Không đúng tuple
<DateRangePicker value={dayjs()} onChange={setRange} />
```

**Đúng:**

```tsx
// ✅ Dayjs instances
<DateRangePicker value={[dayjs("2024-01-01"), dayjs("2024-01-31")]} onChange={setRange} />

// ✅ Null cho trạng thái rỗng
<DateRangePicker value={[null, null]} onChange={setRange} />

// ✅ Chỉ có ngày bắt đầu
<DateRangePicker value={[dayjs(), null]} onChange={setRange} />
```

---

### RULE-DATE-RANGE-PICKER-02: Format có chứa giờ tự kích hoạt DateTime mode — không cần prop riêng

Khi `format` chứa token giờ (`H`, `h`, `m`, `s`, `A`), component tự hiển thị TimeSelect bên cạnh Calendar. Không cần truyền prop `dateTime` riêng trên DateRangePicker.

**Sai:**

```tsx
// ❌ Tự build time picker bên ngoài — logic thừa
<div className={cx("flex gap-[0.5rem]")}>
  <DateRangePicker value={range} onChange={setRange} format="DD/MM/YYYY" />
  <TimeSelect value={time} onChange={setTime} />
</div>
```

**Đúng:**

```tsx
// ✅ Chỉ chọn ngày
<DateRangePicker value={range} onChange={setRange} format="DD/MM/YYYY" />

// ✅ Chọn ngày + giờ — TimeSelect tự hiện
<DateRangePicker value={range} onChange={setRange} format="DD/MM/YYYY HH:mm:ss" />

// ✅ Chọn ngày + giờ 12h
<DateRangePicker value={range} onChange={setRange} format="DD/MM/YYYY hh:mm A" />
```

---

### RULE-DATE-RANGE-PICKER-03: Sử dụng `fromMin`/`fromMax`/`toMin`/`toMax` để giới hạn khoảng ngày — không tự validate bên ngoài

DateRangePicker có 4 props riêng biệt để giới hạn ngày cho input "từ" và input "đến":
- `fromMin` / `fromMax`: Giới hạn ngày tối thiểu / tối đa cho input "từ" (from)
- `toMin` / `toMax`: Giới hạn ngày tối thiểu / tối đa cho input "đến" (to)

Component tự động tính toán `min`/`max` dựa trên input đang focus và giá trị hiện tại. Khi chọn ngày "đến", `min` sẽ tự động là giá trị lớn nhất giữa `toMin` và `value[0]` (ngày bắt đầu). Không tự validate trong `onChange`.

**Sai:**

```tsx
// ❌ Tự validate trong onChange — calendar vẫn cho chọn ngày ngoài khoảng
<DateRangePicker
  value={range}
  onChange={(newRange) => {
    if (newRange[0] && newRange[1] && newRange[0].isAfter(newRange[1])) return;
    setRange(newRange);
  }}
/>

// ❌ Sử dụng min/max chung — không phân biệt from/to
<DateRangePicker
  value={range}
  onChange={setRange}
  min={dayjs().subtract(30, "day")}
  max={dayjs()}
/>
```

**Đúng:**

```tsx
// ✅ Giới hạn ngày bắt đầu từ 30 ngày trước đến hôm nay
<DateRangePicker
  value={range}
  onChange={setRange}
  fromMin={dayjs().subtract(30, "day")}
  fromMax={dayjs()}
/>

// ✅ Giới hạn ngày kết thúc tối đa là 7 ngày sau hôm nay
<DateRangePicker
  value={range}
  onChange={setRange}
  toMax={dayjs().add(7, "day")}
/>

// ✅ Kết hợp cả from và to
<DateRangePicker
  value={range}
  onChange={setRange}
  fromMin={dayjs().subtract(30, "day")}
  fromMax={dayjs()}
  toMin={dayjs()}
  toMax={dayjs().add(30, "day")}
/>
```

---

### RULE-DATE-RANGE-PICKER-04: Component tự động hoán đổi from/to khi from > to — không cần tự xử lý

Khi người dùng nhập tay và blur, nếu `from` lớn hơn `to`, component tự động hoán đổi 2 giá trị. Không cần tự viết logic swap bên ngoài.

**Sai:**

```tsx
// ❌ Tự swap bên ngoài — logic trùng lặp với component
<DateRangePicker
  value={range}
  onChange={(newRange) => {
    if (newRange[0] && newRange[1] && newRange[0].isAfter(newRange[1])) {
      setRange([newRange[1], newRange[0]]);
    } else {
      setRange(newRange);
    }
  }}
/>
```

**Đúng:**

```tsx
// ✅ Component tự động swap khi from > to
<DateRangePicker value={range} onChange={setRange} />
```

---

### RULE-DATE-RANGE-PICKER-05: Sử dụng `renderValue` để tuỳ chỉnh hiển thị — không override input value bên ngoài

`renderValue` nhận `DatePickerValue` (Dayjs | null) và trả về string hiển thị trong input. Hàm này được áp dụng cho cả 2 input (from và to). Không tự override giá trị input qua CSS hoặc wrapper.

**Sai:**

```tsx
// ❌ Tự override hiển thị qua wrapper
<div className={cx("relative")}>
  <DateRangePicker value={range} onChange={setRange} />
  <span className={cx("absolute inset-0")}>{customFormat(range)}</span>
</div>
```

**Đúng:**

```tsx
// ✅ Tuỳ chỉnh hiển thị qua renderValue
<DateRangePicker
  value={range}
  onChange={setRange}
  renderValue={(dateValue) => {
    if (dateValue) return dateValue.format("dddd, DD/MM/YYYY");
    return "";
  }}
/>
```

---

### RULE-DATE-RANGE-PICKER-06: Sử dụng `calendarProps` để truyền config xuống DateRangeCalendar — không tự build calendar

`calendarProps` cho phép tuỳ chỉnh DateRangeCalendar (headerTextFormat, timeSelectProps, disabledKeyboard...). Không tự build calendar component.

**Sai:**

```tsx
// ❌ Tự build calendar bên ngoài
<Popper render={() => <MyCustomCalendar value={range} onChange={setRange} />}>
  {(params) => <div {...params}>...</div>}
</Popper>
```

**Đúng:**

```tsx
// ✅ Tuỳ chỉnh calendar qua calendarProps
<DateRangePicker
  value={range}
  onChange={setRange}
  calendarProps={{
    headerTextFormat: "MMMM YYYY",
    timeSelectProps: { format: "HH:mm" },
  }}
/>
```

---

### RULE-DATE-RANGE-PICKER-07: Input hỗ trợ nhập tay — blur sẽ parse và validate theo `format`

Người dùng có thể gõ trực tiếp vào input from hoặc to. Khi blur, component parse giá trị theo `format` prop bằng strict mode. Nếu invalid, giá trị không thay đổi. Không tự thêm validation logic cho input.

**Sai:**

```tsx
// ❌ Tự validate input — conflict với logic nội bộ
<DateRangePicker
  value={range}
  onChange={setRange}
  onInputChange={(e) => {
    const parsed = dayjs(e.target.value, "DD/MM/YYYY");
    if (parsed.isValid()) setRange([parsed, range[1]]);
  }}
/>
```

**Đúng:**

```tsx
// ✅ Component tự validate khi blur
<DateRangePicker value={range} onChange={setRange} format="DD/MM/YYYY" />
```

---

### RULE-DATE-RANGE-PICKER-08: Sử dụng `helperText` và `error` cho thông báo lỗi — không tự build helper text

Component có sẵn `helperText` và `error` prop. Khi `error=true`, helper text hiển thị màu lỗi và border chuyển sang màu error. Không tự build component hiển thị lỗi bên ngoài.

**Sai:**

```tsx
// ❌ Tự build helper text bên ngoài
<div>
  <DateRangePicker value={range} onChange={setRange} />
  <span className={cx("text-error prose-body2")}>Vui lòng chọn khoảng ngày</span>
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng props có sẵn
<DateRangePicker
  value={range}
  onChange={setRange}
  error={isError}
  helperText="Vui lòng chọn khoảng ngày"
/>
```

---

### RULE-DATE-RANGE-PICKER-09: Sử dụng `getFromRef` và `getToRef` để truy cập input ref — không dùng document.querySelector

Component cung cấp 2 callback `getFromRef` và `getToRef` để lấy ref của 2 input (from và to). Không dùng DOM query để truy cập input.

**Sai:**

```tsx
// ❌ Dùng DOM query
const fromInput = document.querySelector(".stringee-date-range-picker-from-input");
fromInput?.focus();
```

**Đúng:**

```tsx
// ✅ Sử dụng getFromRef / getToRef
const [fromRef, setFromRef] = useState<HTMLInputElement | null>(null);
const [toRef, setToRef] = useState<HTMLInputElement | null>(null);

<DateRangePicker
  value={range}
  onChange={setRange}
  getFromRef={setFromRef}
  getToRef={setToRef}
/>

// Focus vào input from
fromRef?.focus();
```

---

### RULE-DATE-RANGE-PICKER-10: Xoá giá trị bằng icon clear có sẵn — không tự build nút clear

Khi có giá trị và không disabled/readOnly, icon clear (x) tự động hiện khi hover. Click icon clear sẽ đặt cả 2 giá trị về `[null, null]`. Không tự build nút clear bên ngoài.

**Sai:**

```tsx
// ❌ Tự build nút clear bên ngoài
<div className={cx("flex items-center gap-[0.5rem]")}>
  <DateRangePicker value={range} onChange={setRange} />
  <button onClick={() => setRange([null, null])}>x</button>
</div>
```

**Đúng:**

```tsx
// ✅ Icon clear có sẵn, tự động hiện khi hover và có giá trị
<DateRangePicker value={range} onChange={setRange} />
```
