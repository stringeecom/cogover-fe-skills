---
title: Cách sử dụng component TimeRangePicker trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến giá trị thời gian sai format, min/max không hoạt động đúng cho từng input, popper không hiển thị, và clear không hoạt động
tags: time-range-picker, time, range, dayjs, datetime, time-select, ui-kit
---

## Cách sử dụng component TimeRangePicker trong ui-kit

Đường dẫn component: `ui-kit/src/components/TimeRangePicker`

Props chính: `value` ([Dayjs | null, Dayjs | null]), `onChange`, `format`, `showBorderOnBlur`, `timeSelectProps`, `renderValue`, `fullWidth`, `helperText`, `color`, `error`, `className`, `popperProps`, `getFromRef`, `getToRef`, `fromMax`, `fromMin`, `toMax`, `toMin`, `onBlur`. Kế thừa các props từ `React.InputHTMLAttributes<HTMLInputElement>` (trừ `value`, `onChange`, `onBlur`, `max`, `min`).

Giá trị mặc định: `format="HH:mm:ss"`, `showBorderOnBlur=true`, `fullWidth=false`, `timeSelectProps={}`.

---

### RULE-TIME-RANGE-PICKER-01: `value` là tuple `[Dayjs | null, Dayjs | null]` — không truyền string, Date, hay mảng đơn

TimeRangePicker yêu cầu `value` là một tuple gồm 2 phần tử `[from, to]`, mỗi phần tử là `Dayjs` hoặc `null`. Không truyền string, native Date, hay mảng có một phần tử.

**Sai:**

```tsx
// ❌ Truyền mảng string
<TimeRangePicker value={["08:00:00", "17:00:00"]} onChange={setRange} />

// ❌ Truyền native Date
<TimeRangePicker value={[new Date(), new Date()]} onChange={setRange} />

// ❌ Truyền Dayjs đơn lẻ thay vì tuple
<TimeRangePicker value={dayjs()} onChange={setRange} />
```

**Đúng:**

```tsx
// ✅ Tuple Dayjs
<TimeRangePicker value={[dayjs("08:00:00", "HH:mm:ss"), dayjs("17:00:00", "HH:mm:ss")]} onChange={setRange} />

// ✅ Một bên null
<TimeRangePicker value={[dayjs(), null]} onChange={setRange} />

// ✅ Cả hai null
<TimeRangePicker value={[null, null]} onChange={setRange} />
```

---

### RULE-TIME-RANGE-PICKER-02: `onChange` nhận tuple `[Dayjs | null, Dayjs | null]` — state phải lưu đúng kiểu

Callback `onChange` trả về tuple `[Dayjs | null, Dayjs | null]`. State lưu giá trị phải khai báo đúng kiểu để tránh lỗi TypeScript.

**Sai:**

```tsx
// ❌ Khai báo state sai kiểu
const [range, setRange] = useState<Dayjs[]>([]);
<TimeRangePicker value={range} onChange={setRange} />

// ❌ Tách thành 2 state riêng rồi tự merge
const [from, setFrom] = useState<Dayjs | null>(null);
const [to, setTo] = useState<Dayjs | null>(null);
<TimeRangePicker
  value={[from, to]}
  onChange={(val) => {
    setFrom(val[0]);
    setTo(val[1]);
  }}
/>
```

**Đúng:**

```tsx
// ✅ State đúng kiểu tuple
const [range, setRange] = useState<[Dayjs | null, Dayjs | null]>([null, null]);
<TimeRangePicker value={range} onChange={setRange} />
```

---

### RULE-TIME-RANGE-PICKER-03: Sử dụng `fromMin`/`fromMax`/`toMin`/`toMax` để giới hạn thời gian cho từng input — không dùng chung min/max

TimeRangePicker có 4 props riêng biệt để giới hạn thời gian cho input "from" và input "to". Không tự validate trong `onChange` và không truyền `min`/`max` chung.

**Sai:**

```tsx
// ❌ Tự validate trong onChange
<TimeRangePicker
  value={range}
  onChange={(val) => {
    if (val[0] && val[0].isBefore(dayjs("08:00", "HH:mm"))) return;
    setRange(val);
  }}
/>

// ❌ Không có props min/max chung cho cả hai input
<TimeRangePicker value={range} onChange={setRange} min={dayjs("08:00", "HH:mm")} max={dayjs("17:00", "HH:mm")} />
```

**Đúng:**

```tsx
// ✅ Giới hạn riêng cho từng input
<TimeRangePicker
  value={range}
  onChange={setRange}
  fromMin={dayjs("08:00:00", "HH:mm:ss")}
  fromMax={dayjs("12:00:00", "HH:mm:ss")}
  toMin={dayjs("13:00:00", "HH:mm:ss")}
  toMax={dayjs("17:00:00", "HH:mm:ss")}
/>

// ✅ Chỉ giới hạn input "from"
<TimeRangePicker
  value={range}
  onChange={setRange}
  fromMin={dayjs("08:00:00", "HH:mm:ss")}
  fromMax={dayjs("12:00:00", "HH:mm:ss")}
/>
```

---

### RULE-TIME-RANGE-PICKER-04: Sử dụng `format` để thay đổi format hiển thị — mặc định là `"HH:mm:ss"`

`format` quyết định cách hiển thị và parse giá trị trong input. Khi blur, component parse giá trị nhập tay theo `format` bằng strict mode. Không tự format bên ngoài.

**Sai:**

```tsx
// ❌ Tự format bên ngoài rồi truyền vào
<TimeRangePicker
  value={range}
  onChange={setRange}
  renderValue={(val) => val?.format("HH:mm")}
  format="HH:mm:ss"
/>
```

**Đúng:**

```tsx
// ✅ Format mặc định HH:mm:ss
<TimeRangePicker value={range} onChange={setRange} />

// ✅ Format chỉ giờ và phút
<TimeRangePicker value={range} onChange={setRange} format="HH:mm" />

// ✅ Format 12 giờ
<TimeRangePicker value={range} onChange={setRange} format="hh:mm A" />
```

---

### RULE-TIME-RANGE-PICKER-05: Sử dụng `renderValue` để tuỳ chỉnh hiển thị — không override input value bên ngoài

`renderValue` nhận `Dayjs | null` và trả về string hiển thị trong input. Hàm này được áp dụng cho cả hai input (from và to). Không tự override giá trị input qua CSS hoặc wrapper.

**Sai:**

```tsx
// ❌ Tự override hiển thị qua wrapper
<div className={cx("relative")}>
  <TimeRangePicker value={range} onChange={setRange} />
  <span className={cx("absolute inset-0")}>{customFormat(range[0])}</span>
</div>
```

**Đúng:**

```tsx
// ✅ Tuỳ chỉnh hiển thị qua renderValue
<TimeRangePicker
  value={range}
  onChange={setRange}
  renderValue={(val) => {
    if (!val) return "";
    return val.format("HH[h]mm[m]ss[s]");
  }}
/>
```

---

### RULE-TIME-RANGE-PICKER-06: Sử dụng `timeSelectProps` để tuỳ chỉnh TimeSelect bên trong — không tự build time selector

`timeSelectProps` cho phép truyền config xuống component `TimeSelect` nội bộ (height, fullWidth...). Không tự build time selector bên ngoài.

**Sai:**

```tsx
// ❌ Tự build 2 TimeSelect bên ngoài
<div className={cx("flex gap-[0.5rem]")}>
  <TimeSelect value={range[0]} onChange={(v) => setRange([v, range[1]])} />
  <span>-</span>
  <TimeSelect value={range[1]} onChange={(v) => setRange([range[0], v])} />
</div>
```

**Đúng:**

```tsx
// ✅ Tuỳ chỉnh TimeSelect qua timeSelectProps
<TimeRangePicker
  value={range}
  onChange={setRange}
  timeSelectProps={{
    height: 200,
  }}
/>
```

---

### RULE-TIME-RANGE-PICKER-07: Sử dụng `error` + `helperText` cho validation — không tự build error message bên ngoài

`error` bật trạng thái lỗi (border đỏ), `helperText` hiển thị text bên dưới input. Khi `error=true`, helperText tự động có màu lỗi. Không tự wrap error message bên ngoài.

**Sai:**

```tsx
// ❌ Tự build error message bên ngoài
<div>
  <TimeRangePicker value={range} onChange={setRange} />
  {hasError && <span className={cx("text-error prose-caption")}>Thời gian không hợp lệ</span>}
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng error + helperText
<TimeRangePicker
  value={range}
  onChange={setRange}
  error={hasError}
  helperText={hasError ? "Thời gian không hợp lệ" : undefined}
/>
```

---

### RULE-TIME-RANGE-PICKER-08: Sử dụng `getFromRef` và `getToRef` để truy cập input element — không dùng querySelector

`getFromRef` và `getToRef` cung cấp reference đến input element của "from" và "to". Không dùng `querySelector` hay `document.getElementById` để truy cập input.

**Sai:**

```tsx
// ❌ Dùng querySelector để focus input
document.querySelector(".stringee-date-range-picker-from-input")?.focus();
```

**Đúng:**

```tsx
// ✅ Sử dụng getFromRef/getToRef
const fromInputRef = useRef<HTMLInputElement | null>(null);
const toInputRef = useRef<HTMLInputElement | null>(null);

<TimeRangePicker
  value={range}
  onChange={setRange}
  getFromRef={(ref) => { fromInputRef.current = ref; }}
  getToRef={(ref) => { toInputRef.current = ref; }}
/>

// Focus vào input "from"
fromInputRef.current?.focus();
```

---

### RULE-TIME-RANGE-PICKER-09: Clear giá trị bằng cách truyền `[null, null]` — icon clear đã tích hợp sẵn

Component tự hiển thị icon clear (×) khi hover và có giá trị. Click icon sẽ gọi `onChange([null, null])`. Không tự build nút clear bên ngoài. Truyền `disabled` hoặc `readOnly` để ẩn icon clear.

**Sai:**

```tsx
// ❌ Tự build nút clear bên ngoài
<div className={cx("flex items-center gap-[0.5rem]")}>
  <TimeRangePicker value={range} onChange={setRange} />
  <button onClick={() => setRange([null, null])}>×</button>
</div>
```

**Đúng:**

```tsx
// ✅ Icon clear tích hợp sẵn — không cần thêm gì
<TimeRangePicker value={range} onChange={setRange} />

// ✅ Ẩn border và icon clear khi blur
<TimeRangePicker value={range} onChange={setRange} showBorderOnBlur={false} />

// ✅ Programmatic clear
<TimeRangePicker value={range} onChange={setRange} />
<Button onClick={() => setRange([null, null])}>Reset</Button>
```

---

### RULE-TIME-RANGE-PICKER-10: Input hỗ trợ nhập tay — blur sẽ parse và validate theo `format`

Người dùng có thể gõ trực tiếp vào cả hai input (from và to). Khi blur, component parse giá trị theo `format` prop. Nếu invalid, giá trị không thay đổi. Không tự thêm validation logic cho input.

**Sai:**

```tsx
// ❌ Tự validate input — conflict với logic nội bộ
<TimeRangePicker
  value={range}
  onChange={setRange}
  onBlur={() => {
    if (range[0] && range[1] && range[0].isAfter(range[1])) {
      setRange([range[1], range[0]]);
    }
  }}
/>
```

**Đúng:**

```tsx
// ✅ Component tự validate khi blur — chỉ dùng onBlur cho side effects
<TimeRangePicker
  value={range}
  onChange={setRange}
  format="HH:mm:ss"
  onBlur={() => {
    // Side effects: tracking, save draft...
    trackInteraction("time-range-blur");
  }}
/>
```
