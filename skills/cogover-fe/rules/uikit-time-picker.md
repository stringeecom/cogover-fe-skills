---
title: Cách sử dụng component TimePicker trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến giờ hiển thị sai format, popup không mở, min/max không hoạt động, xung đột logic validate
tags: time-picker, time, dayjs, time-select, popper, ui-kit
---

## Cách sử dụng component TimePicker trong ui-kit

Đường dẫn component: `ui-kit/src/components/TimePicker`

Props chính: `value` (Dayjs | null), `onChange`, `format`, `min`, `max`, `renderValue`, `timeSelectProps`, `popperProps`, `helperText`, `hidePopperOnClear`, `showBorderOnBlur`, `className`. Kế thừa các props từ `TextFieldProps` (ngoại trừ `value`, `onChange`, `max`, `min`).

Giá trị mặc định: `format="HH:mm:ss"`, `showBorderOnBlur=true`, `hidePopperOnClear=false`, `timeSelectProps={}`.

---

### RULE-TIME-PICKER-01: `value` là `Dayjs | null` — không truyền Date, string, hay timestamp

TimePicker sử dụng `dayjs` library. `value` phải là instance `Dayjs` hoặc `null`. Không truyền native `Date`, ISO string, hay Unix timestamp trực tiếp.

**Sai:**

```tsx
// ❌ Native Date object
<TimePicker value={new Date()} onChange={setTime} />

// ❌ ISO string
<TimePicker value="14:30:00" onChange={setTime} />

// ❌ Unix timestamp
<TimePicker value={1705276800000} onChange={setTime} />
```

**Đúng:**

```tsx
// ✅ Dayjs instance
<TimePicker value={dayjs()} onChange={setTime} />

// ✅ Dayjs từ string với format
<TimePicker value={dayjs("14:30:00", "HH:mm:ss")} onChange={setTime} />

// ✅ Null cho trạng thái rỗng
<TimePicker value={null} onChange={setTime} />
```

---

### RULE-TIME-PICKER-02: Sử dụng `format` để định dạng hiển thị — mặc định là `"HH:mm:ss"`

`format` quyết định cách giờ hiển thị trong input và cách parse khi người dùng gõ tay. Sử dụng token của dayjs. Không tự format bên ngoài rồi truyền string vào.

**Sai:**

```tsx
// ❌ Tự format rồi truyền string — value phải là Dayjs
<TimePicker value={dayjs().format("HH:mm")} onChange={setTime} />
```

**Đúng:**

```tsx
// ✅ Mặc định HH:mm:ss
<TimePicker value={time} onChange={setTime} />

// ✅ Chỉ giờ và phút
<TimePicker value={time} onChange={setTime} format="HH:mm" />

// ✅ Format 12 giờ
<TimePicker value={time} onChange={setTime} format="hh:mm A" />
```

---

### RULE-TIME-PICKER-03: Sử dụng `min`/`max` là `Dayjs` để giới hạn khoảng thời gian — không tự validate bên ngoài

`min` và `max` nhận `Dayjs | null`. Giá trị này được truyền xuống `TimeSelect` để giới hạn lựa chọn. Không tự validate trong `onChange`.

**Sai:**

```tsx
// ❌ Tự validate trong onChange — TimeSelect vẫn hiển thị các giờ ngoài khoảng
<TimePicker
  value={time}
  onChange={(newTime) => {
    if (newTime && newTime.isAfter(dayjs().hour(17))) return;
    setTime(newTime);
  }}
/>
```

**Đúng:**

```tsx
// ✅ Giới hạn từ 8h sáng đến 5h chiều
<TimePicker
  value={time}
  onChange={setTime}
  min={dayjs().hour(8).minute(0).second(0)}
  max={dayjs().hour(17).minute(0).second(0)}
/>
```

---

### RULE-TIME-PICKER-04: Sử dụng `renderValue` để tuỳ chỉnh hiển thị — không override input value bên ngoài

`renderValue` nhận `Dayjs | null`, trả về string hiển thị trong input khi không focus. Không tự override giá trị input qua CSS hoặc wrapper.

**Sai:**

```tsx
// ❌ Tự override hiển thị qua wrapper
<div className={cx("relative")}>
  <TimePicker value={time} onChange={setTime} />
  <span className={cx("absolute inset-0")}>{customFormat(time)}</span>
</div>
```

**Đúng:**

```tsx
// ✅ Tuỳ chỉnh hiển thị qua renderValue
<TimePicker
  value={time}
  onChange={setTime}
  renderValue={(value) => {
    if (!value) return "";
    return value.format("HH giờ mm phút");
  }}
/>
```

---

### RULE-TIME-PICKER-05: Sử dụng `timeSelectProps` để tuỳ chỉnh TimeSelect — không tự build time select

`timeSelectProps` cho phép truyền config xuống `TimeSelect` (height, fullWidth...). Không tự build time select dropdown bên ngoài.

**Sai:**

```tsx
// ❌ Tự build time select bên ngoài
<Popper render={() => <TimeSelect value={time} onChange={setTime} />}>
  {(params) => <TextField {...params} value={formatTime(time)} />}
</Popper>
```

**Đúng:**

```tsx
// ✅ Tuỳ chỉnh TimeSelect qua timeSelectProps
<TimePicker
  value={time}
  onChange={setTime}
  timeSelectProps={{
    height: 300,
  }}
/>
```

---

### RULE-TIME-PICKER-06: Input hỗ trợ nhập tay — blur sẽ parse và validate theo `format`

Người dùng có thể gõ trực tiếp vào input. Khi blur, component parse giá trị theo `format` prop. Nếu invalid, giá trị không thay đổi. Không tự thêm validation logic cho input.

**Sai:**

```tsx
// ❌ Tự validate input — xung đột với logic nội bộ
<TimePicker
  value={time}
  onChange={setTime}
  onInputChange={(e) => {
    const parsed = dayjs(e.target.value, "HH:mm:ss");
    if (parsed.isValid()) setTime(parsed);
  }}
/>
```

**Đúng:**

```tsx
// ✅ Component tự validate khi blur
<TimePicker value={time} onChange={setTime} format="HH:mm:ss" />

// ✅ onInputChange chỉ dùng cho side effects (tracking, logging)
<TimePicker
  value={time}
  onChange={setTime}
  onInputChange={(e) => trackInputChange(e.target.value)}
/>
```

---

### RULE-TIME-PICKER-07: Sử dụng `hidePopperOnClear` để đóng popup khi xoá giá trị

Mặc định khi nhấn nút clear (×), popup TimeSelect vẫn mở. Truyền `hidePopperOnClear={true}` nếu muốn popup tự đóng khi xoá giá trị.

**Sai:**

```tsx
// ❌ Tự xử lý đóng popup qua ref — không cần thiết
const popperRef = useRef();
<TimePicker
  value={time}
  onChange={(val) => {
    if (!val) popperRef.current?.hide();
    setTime(val);
  }}
/>
```

**Đúng:**

```tsx
// ✅ Mặc định — popup vẫn mở khi clear
<TimePicker value={time} onChange={setTime} />

// ✅ Đóng popup khi clear
<TimePicker value={time} onChange={setTime} hidePopperOnClear />
```

---

### RULE-TIME-PICKER-08: Sử dụng `helperText` kết hợp `error` để hiển thị thông báo lỗi

`helperText` hiển thị text phía dưới input. Khi kết hợp với `error={true}`, text sẽ hiển thị màu lỗi. Không tự build error message bên ngoài.

**Sai:**

```tsx
// ❌ Tự build error message bên ngoài
<div>
  <TimePicker value={time} onChange={setTime} />
  {hasError && <span className={cx("text-error")}>Vui lòng chọn giờ</span>}
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng helperText + error
<TimePicker
  value={time}
  onChange={setTime}
  error={hasError}
  helperText={hasError ? "Vui lòng chọn giờ" : undefined}
/>
```

---

### RULE-TIME-PICKER-09: Sử dụng `popperProps` để tuỳ chỉnh Popper — không tự wrap Popper bên ngoài

`popperProps` cho phép tuỳ chỉnh vị trí, offset và hành vi của popup. Không tự wrap `Popper` bên ngoài TimePicker.

**Sai:**

```tsx
// ❌ Tự wrap Popper bên ngoài — tạo nested popup
<Popper placement="top-start">
  <TimePicker value={time} onChange={setTime} />
</Popper>
```

**Đúng:**

```tsx
// ✅ Tuỳ chỉnh popup qua popperProps
<TimePicker
  value={time}
  onChange={setTime}
  popperProps={{
    placement: "top-start",
  }}
/>
```

---

### RULE-TIME-PICKER-10: Không tự build icon — component đã tích hợp sẵn icon đồng hồ và nút clear

TimePicker tự hiển thị icon đồng hồ (khi `showBorderOnBlur={true}`) và nút clear (×) khi có giá trị. Không tự thêm icon hoặc nút clear bên ngoài.

**Sai:**

```tsx
// ❌ Tự thêm icon và nút clear
<div className={cx("flex items-center gap-[0.5rem]")}>
  <TimePicker value={time} onChange={setTime} />
  <FontAwesomeIcon icon={faClock} />
  <button onClick={() => setTime(null)}>×</button>
</div>
```

**Đúng:**

```tsx
// ✅ Icon đồng hồ và nút clear đã tích hợp sẵn
<TimePicker value={time} onChange={setTime} />

// ✅ Ẩn border và icon khi blur
<TimePicker value={time} onChange={setTime} showBorderOnBlur={false} />
```
