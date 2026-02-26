---
title: Cách sử dụng component DatePicker trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến ngày hiển thị sai format, calendar không mở, min/max không hoạt động, time ago và relative date conflict
tags: date-picker, date, calendar, time, dayjs, datetime, time-ago, relative, ui-kit
---

## Cách sử dụng component DatePicker trong ui-kit

Đường dẫn component: `ui-kit/src/components/DatePicker`
DatePickerRelative: `ui-kit/src/components/DatePicker/DatePickerRelative`

Props chính: `value` (Dayjs | null), `onChange`, `format`, `min`, `max`, `height`, `clearable`, `showEndIcon`, `showBorderOnBlur`, `hidePopperOnClear`, `locale`, `calendarProps`, `popperProps`, `renderValue`, `useTimeAgoValue`, `timeAgoValue`, `timeAgoOptions`, `onChangeTimeAgoValue`, `useRelativeValue`, `relativeValue`, `customTab`. Kế thừa các props từ `TextFieldProps`.

Giá trị mặc định: `format="DD/MM/YYYY"`, `height=40`, `clearable=true`, `showEndIcon=true`, `showBorderOnBlur=true`, `hidePopperOnClear=false`, `timeAgoOptions=[60000, 900000, 1800000, 2700000, 3600000]`.

---

### RULE-DATE-PICKER-01: `value` là `Dayjs | null` — không truyền Date, string, hay timestamp

DatePicker sử dụng `dayjs` library. `value` phải là instance `Dayjs` hoặc `null`. Không truyền native `Date`, ISO string, hay Unix timestamp trực tiếp.

**Sai:**

```tsx
// ❌ Native Date object
<DatePicker value={new Date()} onChange={setDate} />

// ❌ ISO string
<DatePicker value="2024-01-15" onChange={setDate} />

// ❌ Unix timestamp
<DatePicker value={1705276800000} onChange={setDate} />
```

**Đúng:**

```tsx
// ✅ Dayjs instance
<DatePicker value={dayjs()} onChange={setDate} />

// ✅ Dayjs từ string
<DatePicker value={dayjs("2024-01-15")} onChange={setDate} />

// ✅ Null cho trạng thái rỗng
<DatePicker value={null} onChange={setDate} />
```

---

### RULE-DATE-PICKER-02: Format có chứa giờ tự kích hoạt DateTime mode — không cần prop riêng

Khi `format` chứa token giờ (`H`, `h`, `m`, `s`, `A`), component tự hiển thị TimeSelect bên cạnh Calendar. Không cần truyền prop `dateTime` riêng trên DatePicker.

**Sai:**

```tsx
// ❌ Tự build time picker bên ngoài — logic thừa
<div className={cx("flex gap-[0.5rem]")}>
  <DatePicker value={date} onChange={setDate} format="DD/MM/YYYY" />
  <TimeSelect value={time} onChange={setTime} />
</div>
```

**Đúng:**

```tsx
// ✅ Chỉ chọn ngày
<DatePicker value={date} onChange={setDate} format="DD/MM/YYYY" />

// ✅ Chọn ngày + giờ — TimeSelect tự hiện
<DatePicker value={date} onChange={setDate} format="DD/MM/YYYY HH:mm:ss" />

// ✅ Chọn ngày + giờ 12h
<DatePicker value={date} onChange={setDate} format="DD/MM/YYYY hh:mm A" />
```

---

### RULE-DATE-PICKER-03: Sử dụng `min`/`max` là `Dayjs` để giới hạn khoảng ngày — không tự validate bên ngoài

`min` và `max` nhận `Dayjs` instance. Calendar tự disable các ngày ngoài khoảng và keyboard navigation tự dừng tại biên. Không tự validate trong `onChange`.

**Sai:**

```tsx
// ❌ Tự validate trong onChange — calendar vẫn cho chọn ngày ngoài khoảng
<DatePicker
  value={date}
  onChange={(newDate) => {
    if (newDate && newDate.isAfter(dayjs())) return;
    setDate(newDate);
  }}
/>
```

**Đúng:**

```tsx
// ✅ Chỉ cho chọn từ 30 ngày trước đến hôm nay
<DatePicker
  value={date}
  onChange={setDate}
  min={dayjs().subtract(30, "day")}
  max={dayjs()}
/>
```

---

### RULE-DATE-PICKER-04: Sử dụng `clearable` để cho phép xoá giá trị — mặc định đã bật

`clearable=true` mặc định — icon clear (×) hiện khi hover và có giá trị. Truyền `clearable={false}` khi field bắt buộc. Không tự build nút clear.

**Sai:**

```tsx
// ❌ Tự build nút clear bên ngoài
<div className={cx("flex items-center gap-[0.5rem]")}>
  <DatePicker value={date} onChange={setDate} />
  <button onClick={() => setDate(null)}>×</button>
</div>
```

**Đúng:**

```tsx
// ✅ Clearable mặc định
<DatePicker value={date} onChange={setDate} />

// ✅ Tắt clear cho field bắt buộc
<DatePicker value={date} onChange={setDate} clearable={false} />
```

---

### RULE-DATE-PICKER-05: Sử dụng `useTimeAgoValue` với `timeAgoValue` + `onChangeTimeAgoValue` cho preset time ago

Khi bật `useTimeAgoValue`, component hiển thị danh sách presets (1 phút, 15 phút...). Chọn time ago sẽ clear `value` (ngày) và ngược lại. Ba props phải đi cùng nhau.

**Sai:**

```tsx
// ❌ Thiếu onChangeTimeAgoValue — time ago không hoạt động
<DatePicker
  value={date}
  onChange={setDate}
  useTimeAgoValue
  timeAgoValue={timeAgo}
/>

// ❌ Tự build time ago presets bên ngoài
<div>
  <DatePicker value={date} onChange={setDate} />
  <button onClick={() => setTimeAgo(60000)}>1 phút trước</button>
</div>
```

**Đúng:**

```tsx
// ✅ Time ago presets đầy đủ
<DatePicker
  value={date}
  onChange={setDate}
  useTimeAgoValue
  timeAgoValue={timeAgo}
  onChangeTimeAgoValue={setTimeAgo}
/>

// ✅ Tuỳ chỉnh time ago options (ms)
<DatePicker
  value={date}
  onChange={setDate}
  useTimeAgoValue
  timeAgoValue={timeAgo}
  onChangeTimeAgoValue={setTimeAgo}
  timeAgoOptions={[300000, 600000, 1800000, 3600000, 7200000]}
/>
```

---

### RULE-DATE-PICKER-06: Sử dụng `renderValue` để tuỳ chỉnh hiển thị — không override input value bên ngoài

`renderValue` nhận cả `value` và `timeAgoValue`, trả về string hiển thị trong input. Không tự override giá trị input qua CSS hoặc wrapper.

**Sai:**

```tsx
// ❌ Tự override hiển thị qua wrapper
<div className={cx("relative")}>
  <DatePicker value={date} onChange={setDate} />
  <span className={cx("absolute inset-0")}>{customFormat(date)}</span>
</div>
```

**Đúng:**

```tsx
// ✅
<DatePicker
  value={date}
  onChange={setDate}
  renderValue={(dateValue, timeAgoValue) => {
    if (timeAgoValue) return `${timeAgoValue / 60000} phút trước`;
    if (dateValue) return dateValue.format("dddd, DD/MM/YYYY");
    return "";
  }}
/>
```

---

### RULE-DATE-PICKER-07: Sử dụng `calendarProps` để truyền config xuống Calendar — không tự build calendar

`calendarProps` cho phép tuỳ chỉnh Calendar (headerTextFormat, timeSelectProps, disabledKeyboard...). Không tự build calendar component.

**Sai:**

```tsx
// ❌ Tự build calendar bên ngoài
<Popper render={() => <MyCustomCalendar value={date} onChange={setDate} />}>
  {(params) => <TextField {...params} value={formatDate(date)} />}
</Popper>
```

**Đúng:**

```tsx
// ✅ Tuỳ chỉnh calendar qua calendarProps
<DatePicker
  value={date}
  onChange={setDate}
  calendarProps={{
    headerTextFormat: "MMMM YYYY",
    disabledKeyboard: true,
  }}
/>
```

---

### RULE-DATE-PICKER-08: Input hỗ trợ nhập tay — blur sẽ parse và validate theo `format`

Người dùng có thể gõ trực tiếp vào input. Khi blur, component parse giá trị theo `format` prop bằng strict mode. Nếu invalid, giá trị không thay đổi. Không tự thêm validation logic cho input.

**Sai:**

```tsx
// ❌ Tự validate input — conflict với logic nội bộ
<DatePicker
  value={date}
  onChange={setDate}
  onInputChange={(e) => {
    const parsed = dayjs(e.target.value, "DD/MM/YYYY");
    if (parsed.isValid()) setDate(parsed);
  }}
/>
```

**Đúng:**

```tsx
// ✅ Component tự validate khi blur
<DatePicker value={date} onChange={setDate} format="DD/MM/YYYY" />

// ✅ onInputChange chỉ dùng cho side effects (tracking, logging)
<DatePicker
  value={date}
  onChange={setDate}
  onInputChange={(e) => trackInputChange(e.target.value)}
/>
```
