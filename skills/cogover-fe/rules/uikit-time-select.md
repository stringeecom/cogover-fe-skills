---
title: Cách sử dụng component TimeSelect trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến thời gian hiển thị sai format, min/max không hoạt động, danh sách cuộn không đúng vị trí, hoặc build thừa logic chọn giờ bên ngoài
tags: time-select, time, dayjs, datetime, ui-kit
---

## Cách sử dụng component TimeSelect trong ui-kit

Đường dẫn component: `ui-kit/src/components/TimeSelect`

Props chính: `value` (Dayjs | null), `onChange`, `format`, `fullWidth`, `height`, `min`, `max`.

Giá trị mặc định: `format="HH:mm:ss"`, `fullWidth=false`, `height=300`.

Component hiển thị danh sách các mốc thời gian cách nhau 15 phút (96 mốc trong ngày: 00:00, 00:15, 00:30, ..., 23:45). Khi mount, danh sách tự cuộn đến mốc đang được chọn.

---

### RULE-TIME-SELECT-01: `value` là `Dayjs | null` — không truyền Date, string, hay timestamp

TimeSelect sử dụng `dayjs` library. `value` phải là instance `Dayjs` hoặc `null`. Không truyền native `Date`, ISO string, hay Unix timestamp trực tiếp.

**Sai:**

```tsx
// ❌ Native Date object
<TimeSelect value={new Date()} onChange={setTime} />

// ❌ String
<TimeSelect value="14:30:00" onChange={setTime} />

// ❌ Unix timestamp
<TimeSelect value={1705276800000} onChange={setTime} />
```

**Đúng:**

```tsx
// ✅ Dayjs instance
<TimeSelect value={dayjs()} onChange={setTime} />

// ✅ Dayjs từ string
<TimeSelect value={dayjs("2024-01-15 14:30:00")} onChange={setTime} />

// ✅ Null cho trạng thái chưa chọn
<TimeSelect value={null} onChange={setTime} />
```

---

### RULE-TIME-SELECT-02: Sử dụng `format` để thay đổi hiển thị — không tự format bên ngoài

`format` quyết định cách hiển thị nhãn từng mốc thời gian và cách so sánh giá trị đang chọn. Mặc định là `"HH:mm:ss"`. Không tự format giá trị bên ngoài rồi hiển thị.

**Sai:**

```tsx
// ❌ Tự format bên ngoài — không ảnh hưởng đến nhãn trong danh sách
<TimeSelect
  value={time}
  onChange={(val) => {
    setDisplayText(val.format("hh:mm A"));
    setTime(val);
  }}
/>
```

**Đúng:**

```tsx
// ✅ Hiển thị giờ:phút:giây (mặc định)
<TimeSelect value={time} onChange={setTime} />

// ✅ Hiển thị giờ:phút không có giây
<TimeSelect value={time} onChange={setTime} format="HH:mm" />

// ✅ Hiển thị 12 giờ với AM/PM
<TimeSelect value={time} onChange={setTime} format="hh:mm A" />
```

---

### RULE-TIME-SELECT-03: Sử dụng `min`/`max` là `Dayjs` để giới hạn khoảng thời gian — không tự validate bên ngoài

`min` và `max` nhận `Dayjs | null`. Component tự disable các mốc ngoài khoảng (hiển thị mờ và không cho click). Hỗ trợ cả khoảng thời gian qua đêm (min > max). Không tự validate trong `onChange`.

**Sai:**

```tsx
// ❌ Tự validate trong onChange — danh sách vẫn cho chọn mốc ngoài khoảng
<TimeSelect
  value={time}
  onChange={(newTime) => {
    if (newTime.hour() < 8 || newTime.hour() > 17) return;
    setTime(newTime);
  }}
/>
```

**Đúng:**

```tsx
// ✅ Giới hạn từ 08:00 đến 17:00
<TimeSelect
  value={time}
  onChange={setTime}
  min={dayjs().hour(8).minute(0).second(0)}
  max={dayjs().hour(17).minute(0).second(0)}
/>

// ✅ Khoảng qua đêm: từ 22:00 đến 06:00 sáng hôm sau
<TimeSelect
  value={time}
  onChange={setTime}
  min={dayjs().hour(22).minute(0).second(0)}
  max={dayjs().hour(6).minute(0).second(0)}
/>

// ✅ Chỉ giới hạn min
<TimeSelect
  value={time}
  onChange={setTime}
  min={dayjs().hour(9).minute(0).second(0)}
/>
```

---

### RULE-TIME-SELECT-04: Sử dụng `height` để thay đổi chiều cao danh sách — không override CSS bên ngoài

`height` nhận số (đơn vị pixel), mặc định `300`. Component sử dụng CSS variable `--height` nội bộ. Không tự override chiều cao qua CSS hoặc style bên ngoài.

**Sai:**

```tsx
// ❌ Override CSS bên ngoài — conflict với logic nội bộ
<div style={{ height: 200, overflow: "hidden" }}>
  <TimeSelect value={time} onChange={setTime} />
</div>
```

**Đúng:**

```tsx
// ✅ Chiều cao mặc định 300px
<TimeSelect value={time} onChange={setTime} />

// ✅ Tuỳ chỉnh chiều cao
<TimeSelect value={time} onChange={setTime} height={200} />
```

---

### RULE-TIME-SELECT-05: Sử dụng `fullWidth` để mở rộng chiều ngang — không tự set width bên ngoài

Mặc định component có chiều rộng cố định `150px`. Truyền `fullWidth` để component chiếm toàn bộ chiều rộng container. Không tự set width qua CSS bên ngoài.

**Sai:**

```tsx
// ❌ Override width bên ngoài
<div style={{ width: "100%" }}>
  <TimeSelect value={time} onChange={setTime} />
</div>
```

**Đúng:**

```tsx
// ✅ Chiều rộng mặc định 150px
<TimeSelect value={time} onChange={setTime} />

// ✅ Chiếm toàn bộ chiều rộng container
<TimeSelect value={time} onChange={setTime} fullWidth />
```

---

### RULE-TIME-SELECT-06: Không tự build danh sách chọn giờ — sử dụng TimeSelect có sẵn

TimeSelect đã xử lý đầy đủ: tạo 96 mốc thời gian (cách 15 phút), auto-scroll đến mốc đang chọn, disable mốc ngoài khoảng min/max, hỗ trợ khoảng qua đêm. Không tự build component tương tự.

**Sai:**

```tsx
// ❌ Tự build danh sách chọn giờ — thiếu logic auto-scroll, disable, qua đêm
const hours = Array.from({ length: 24 }, (_, i) => i);
const minutes = [0, 15, 30, 45];

<div className={cx("overflow-auto h-[18.75rem]")}>
  {hours.map((h) =>
    minutes.map((m) => (
      <div key={`${h}:${m}`} onClick={() => setTime(dayjs().hour(h).minute(m))}>
        {`${h}:${String(m).padStart(2, "0")}`}
      </div>
    ))
  )}
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng TimeSelect có sẵn
<TimeSelect value={time} onChange={setTime} />
```

---

### RULE-TIME-SELECT-07: Khi cần chọn cả ngày và giờ — ưu tiên dùng DatePicker với format có token giờ thay vì kết hợp riêng TimeSelect

DatePicker tự hiển thị TimeSelect bên trong khi `format` chứa token giờ. Không cần ghép DatePicker và TimeSelect thủ công.

**Sai:**

```tsx
// ❌ Ghép thủ công DatePicker + TimeSelect — phải tự merge giá trị ngày và giờ
<div className={cx("flex gap-[0.5rem]")}>
  <DatePicker value={date} onChange={setDate} format="DD/MM/YYYY" />
  <TimeSelect value={time} onChange={setTime} />
</div>
```

**Đúng:**

```tsx
// ✅ DatePicker tự tích hợp TimeSelect khi format có token giờ
<DatePicker value={dateTime} onChange={setDateTime} format="DD/MM/YYYY HH:mm:ss" />

// ✅ Chỉ dùng TimeSelect độc lập khi không cần chọn ngày
<TimeSelect value={time} onChange={setTime} />
```
