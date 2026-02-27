## Cách sử dụng component TimeSelect trong ui-kit

Đường dẫn: `ui-kit/src/components/TimeSelect`

Props: `value` (Dayjs|null), `onChange`, `format` (default "HH:mm:ss"), `fullWidth` (default false), `height` (default 300, px), `min` (Dayjs|null), `max` (Dayjs|null).

Hiển thị 96 mốc thời gian cách 15 phút (00:00 → 23:45). Tự scroll đến mốc đang chọn khi mount.

```tsx
// Cơ bản
<TimeSelect value={dayjs()} onChange={setTime} />
<TimeSelect value={null} onChange={setTime} />       // chưa chọn

// Tuỳ chỉnh format (ảnh hưởng nhãn trong danh sách)
<TimeSelect value={time} onChange={setTime} format="HH:mm" />
<TimeSelect value={time} onChange={setTime} format="hh:mm A" />

// Giới hạn khoảng thời gian (disable các mốc ngoài khoảng)
<TimeSelect
  value={time} onChange={setTime}
  min={dayjs().hour(8).minute(0).second(0)}
  max={dayjs().hour(17).minute(0).second(0)}
/>
// Khoảng qua đêm (min > max được hỗ trợ)
<TimeSelect value={time} onChange={setTime} min={dayjs().hour(22).minute(0)} max={dayjs().hour(6).minute(0)} />

// Chiều cao và chiều rộng
<TimeSelect value={time} onChange={setTime} height={200} />
<TimeSelect value={time} onChange={setTime} fullWidth />
```

**QUAN TRỌNG — Khi cần chọn ngày + giờ, dùng DatePicker thay vì ghép riêng:**
```tsx
// DON'T: ghép thủ công DatePicker + TimeSelect (phải tự merge ngày giờ)
// DO: DatePicker tự tích hợp TimeSelect khi format có token giờ
<DatePicker value={dateTime} onChange={setDateTime} format="DD/MM/YYYY HH:mm:ss" />

// Chỉ dùng TimeSelect độc lập khi không cần chọn ngày
<TimeSelect value={time} onChange={setTime} />
```

**DO:** Dùng `Dayjs|null` cho `value`, `min`, `max`. Dùng `format` để thay đổi nhãn danh sách. Dùng `min`/`max` để disable mốc ngoài khoảng. Dùng `height` thay vì override CSS. Dùng `fullWidth` thay vì set width bên ngoài. Ưu tiên DatePicker khi cần chọn cả ngày lẫn giờ.

**DON'T:** Truyền `Date`, string, timestamp vào `value`. Tự validate trong `onChange`. Tự build danh sách chọn giờ. Override chiều cao/rộng bằng CSS bên ngoài. Tự format giá trị bên ngoài rồi hiển thị.
