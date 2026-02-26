---
title: Cách sử dụng component Calendar trong ui-kit
impact: HIGH
impactDescription: Sử dụng Calendar sai dẫn đến ngày hiển thị sai, keyboard navigation không hoạt động, min/max không giới hạn đúng, và datetime mode bị lỗi
tags: calendar, date, dayjs, datetime, time-select, year, keyboard, ui-kit
---

## Cách sử dụng component Calendar trong ui-kit

Đường dẫn component: `ui-kit/src/components/Calendar`

Props: `value` (Dayjs | null), `onChange`, `headerTextFormat`, `disabledKeyboard`, `min` (Dayjs), `max` (Dayjs), `dateTime`, `timeSelectProps`. Kế thừa các props từ `React.HTMLAttributes<HTMLDivElement>` (trừ `onChange`) và hỗ trợ `ref` forwarding.

Giá trị mặc định: `headerTextFormat="MM/YYYY"`, `disabledKeyboard=false`, `dateTime=false`.

Callback `onChange` nhận 2 tham số: `(value: Dayjs, options: { isChangeTime: boolean })`. Thuộc tính `isChangeTime` cho biết thay đổi đến từ TimeSelect (`true`) hay từ việc chọn ngày (`false`).

---

### RULE-CALENDAR-01: `value` là `Dayjs | null` — không truyền Date, string, hay timestamp

Calendar sử dụng `dayjs` library. `value` phải là instance `Dayjs` hoặc `null`. Không truyền native `Date`, ISO string, hay Unix timestamp trực tiếp.

**Sai:**

```tsx
// ❌ Native Date object
<Calendar value={new Date()} onChange={handleChange} />

// ❌ ISO string
<Calendar value="2024-01-15" onChange={handleChange} />

// ❌ Unix timestamp
<Calendar value={1705276800000} onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅ Dayjs instance
<Calendar value={dayjs()} onChange={handleChange} />

// ✅ Dayjs từ string
<Calendar value={dayjs("2024-01-15")} onChange={handleChange} />

// ✅ Null cho trạng thái chưa chọn ngày
<Calendar value={null} onChange={handleChange} />
```

---

### RULE-CALENDAR-02: `onChange` nhận callback với 2 tham số — phải xử lý `options.isChangeTime`

Callback `onChange` luôn nhận `(value: Dayjs, options: { isChangeTime: boolean })`. Khi `isChangeTime=true`, thay đổi đến từ TimeSelect (chỉ thay đổi giờ). Khi `isChangeTime=false`, thay đổi đến từ việc chọn ngày. Không bỏ qua tham số `options`.

**Sai:**

```tsx
// ❌ Bỏ qua tham số options
<Calendar
  value={date}
  onChange={(newDate) => {
    setDate(newDate);
  }}
/>
```

**Đúng:**

```tsx
// ✅ Xử lý đầy đủ options
<Calendar
  value={date}
  onChange={(newDate, { isChangeTime }) => {
    setDate(newDate);
    if (!isChangeTime) {
      // Xử lý logic khi người dùng chọn ngày mới
      closePopper();
    }
  }}
/>
```

---

### RULE-CALENDAR-03: Sử dụng `min`/`max` là `Dayjs` để giới hạn khoảng ngày — không tự validate bên ngoài

`min` và `max` nhận `Dayjs` instance. Calendar tự disable các ngày ngoài khoảng, keyboard navigation tự dừng tại biên, và year options tự ẩn các năm ngoài khoảng. Không tự validate trong `onChange`.

**Sai:**

```tsx
// ❌ Tự validate trong onChange — calendar vẫn hiển thị ngày ngoài khoảng
<Calendar
  value={date}
  onChange={(newDate, options) => {
    if (newDate.isAfter(dayjs())) return;
    setDate(newDate);
  }}
/>
```

**Đúng:**

```tsx
// ✅ Calendar tự disable ngày ngoài khoảng
<Calendar
  value={date}
  onChange={(newDate, options) => setDate(newDate)}
  min={dayjs().subtract(30, "day")}
  max={dayjs()}
/>
```

---

### RULE-CALENDAR-04: Bật `dateTime` để hiển thị TimeSelect — không tự build time picker bên ngoài

Khi `dateTime={true}`, Calendar tự hiển thị TimeSelect bên cạnh lưới ngày. Khi chọn ngày trong datetime mode, giờ/phút/giây được giữ nguyên từ `value` hiện tại. Không tự build time picker riêng.

**Sai:**

```tsx
// ❌ Tự build time picker bên ngoài — logic thừa
<div className={cx("flex gap-[0.5rem]")}>
  <Calendar value={date} onChange={handleChange} />
  <TimeSelect value={date} onChange={handleTimeChange} />
</div>
```

**Đúng:**

```tsx
// ✅ Chỉ chọn ngày
<Calendar value={date} onChange={handleChange} />

// ✅ Chọn ngày + giờ — TimeSelect tự hiện bên cạnh
<Calendar value={date} onChange={handleChange} dateTime />

// ✅ Tuỳ chỉnh TimeSelect qua timeSelectProps
<Calendar
  value={date}
  onChange={handleChange}
  dateTime
  timeSelectProps={{ format: "HH:mm:ss" }}
/>
```

---

### RULE-CALENDAR-05: Sử dụng `headerTextFormat` để tuỳ chỉnh hiển thị tháng/năm trên header — không tự build header

`headerTextFormat` nhận format string của dayjs (mặc định `"MM/YYYY"`). Header tự hiển thị nút điều hướng tháng trước/sau và nút chọn năm. Không tự build header navigation.

**Sai:**

```tsx
// ❌ Tự build header navigation bên ngoài
<div>
  <div className={cx("flex justify-between")}>
    <button onClick={() => setMonth(month - 1)}>Prev</button>
    <span>{dayjs().month(month).format("MMMM YYYY")}</span>
    <button onClick={() => setMonth(month + 1)}>Next</button>
  </div>
  <Calendar value={date} onChange={handleChange} />
</div>
```

**Đúng:**

```tsx
// ✅ Mặc định hiển thị MM/YYYY
<Calendar value={date} onChange={handleChange} />

// ✅ Tuỳ chỉnh format header
<Calendar value={date} onChange={handleChange} headerTextFormat="MMMM YYYY" />
```

---

### RULE-CALENDAR-06: Sử dụng `disabledKeyboard` khi không muốn keyboard navigation — không tự chặn event

Calendar mặc định hỗ trợ keyboard navigation: ArrowLeft/Right di chuyển theo ngày, ArrowUp/Down di chuyển theo tuần, Enter chọn ngày đang focus. Dùng `disabledKeyboard={true}` để tắt. Không tự chặn keyboard event bên ngoài.

**Sai:**

```tsx
// ❌ Tự chặn keyboard event bên ngoài
<div
  onKeyDown={(e) => {
    if (["ArrowLeft", "ArrowRight", "ArrowUp", "ArrowDown"].includes(e.key)) {
      e.preventDefault();
    }
  }}
>
  <Calendar value={date} onChange={handleChange} />
</div>
```

**Đúng:**

```tsx
// ✅ Tắt keyboard navigation qua prop
<Calendar value={date} onChange={handleChange} disabledKeyboard />

// ✅ Mặc định có keyboard navigation
<Calendar value={date} onChange={handleChange} />
```

---

### RULE-CALENDAR-07: `timeSelectProps` chỉ truyền khi `dateTime={true}` — không truyền khi không dùng datetime mode

`timeSelectProps` dùng để tuỳ chỉnh TimeSelect component (format, fullWidth). Chỉ có hiệu lực khi `dateTime={true}`. Không truyền `timeSelectProps` khi không bật datetime mode vì sẽ không có tác dụng.

**Sai:**

```tsx
// ❌ Truyền timeSelectProps nhưng không bật dateTime — không có tác dụng
<Calendar
  value={date}
  onChange={handleChange}
  timeSelectProps={{ format: "HH:mm" }}
/>
```

**Đúng:**

```tsx
// ✅ Truyền timeSelectProps cùng với dateTime
<Calendar
  value={date}
  onChange={handleChange}
  dateTime
  timeSelectProps={{ format: "HH:mm" }}
/>
```

---

### RULE-CALENDAR-08: Calendar tự điều hướng đến tháng/năm của `value` — không cần tự quản lý tháng hiện tại

Khi `value` thay đổi, Calendar tự động điều hướng đến tháng/năm tương ứng. Không cần tự quản lý state tháng hiện tại để sync với value.

**Sai:**

```tsx
// ❌ Tự quản lý tháng hiện tại — conflict với logic nội bộ
const [currentMonth, setCurrentMonth] = useState(dayjs());

<Calendar
  value={date}
  onChange={(newDate, options) => {
    setDate(newDate);
    setCurrentMonth(newDate);
  }}
/>
```

**Đúng:**

```tsx
// ✅ Calendar tự sync tháng hiện tại với value
<Calendar
  value={date}
  onChange={(newDate, options) => setDate(newDate)}
/>
```

## Keyboard Navigation

| Phím | Chế độ xem ngày | Chế độ xem năm |
|------|------------------|-----------------|
| `ArrowRight` | Di chuyển sang ngày tiếp theo | Di chuyển sang năm tiếp theo |
| `ArrowLeft` | Di chuyển về ngày trước đó | Di chuyển về năm trước đó |
| `ArrowDown` | Di chuyển xuống 7 ngày (1 tuần) | Di chuyển xuống 3 năm (1 hàng) |
| `ArrowUp` | Di chuyển lên 7 ngày (1 tuần) | Di chuyển lên 3 năm (1 hàng) |
| `Enter` | Chọn ngày đang focus | Chọn năm đang focus |

## Giới hạn năm mặc định

| Thuộc tính | Giá trị |
|------------|---------|
| `defaultMinYear` | 1500 |
| `defaultMaxYear` | 2200 |
