## Cách sử dụng component DatePicker trong ui-kit

Đường dẫn: `ui-kit/src/components/DatePicker`
DatePickerRelative: `ui-kit/src/components/DatePicker/DatePickerRelative`

Props: `value` (Dayjs|null), `onChange`, `format` (default "DD/MM/YYYY"), `min`, `max`, `height` (default 40), `clearable` (default true), `showEndIcon` (default true), `showBorderOnBlur` (default true), `hidePopperOnClear` (default false), `locale`, `calendarProps`, `popperProps`, `renderValue`, `useTimeAgoValue`, `timeAgoValue`, `timeAgoOptions` (default [60000,900000,1800000,2700000,3600000]), `onChangeTimeAgoValue`, `useRelativeValue`, `relativeValue`, `customTab`. Kế thừa `TextFieldProps`.

```tsx
// Chỉ chọn ngày
<DatePicker value={dayjs()} onChange={setDate} />

// Chọn ngày + giờ (format có token giờ → tự kích hoạt TimeSelect)
<DatePicker value={date} onChange={setDate} format="DD/MM/YYYY HH:mm:ss" />

// Giới hạn khoảng ngày
<DatePicker value={date} onChange={setDate} min={dayjs().subtract(30, "day")} max={dayjs()} />

// Time ago presets (3 props phải đi cùng nhau)
<DatePicker value={date} onChange={setDate} useTimeAgoValue timeAgoValue={timeAgo} onChangeTimeAgoValue={setTimeAgo} />

// Custom hiển thị
<DatePicker
  value={date} onChange={setDate}
  renderValue={(dateValue, timeAgoValue) => timeAgoValue ? `${timeAgoValue / 60000} phút trước` : dateValue?.format("dddd, DD/MM/YYYY") ?? ""}
/>

// Tuỳ chỉnh calendar
<DatePicker value={date} onChange={setDate} calendarProps={{ headerTextFormat: "MMMM YYYY", disabledKeyboard: true }} />
```

**DO:** Dùng `Dayjs|null` cho `value`, `min`, `max`. Dùng format với token giờ để kích hoạt DateTime mode. Dùng `renderValue` để tuỳ chỉnh hiển thị. Dùng `calendarProps` để config calendar. Truyền đủ 3 props khi dùng `useTimeAgoValue`.

**DON'T:** Truyền `Date`, string, timestamp vào `value`. Tự build time picker bên ngoài khi cần datetime. Tự validate trong `onChange`. Tự build clear button (clearable mặc định bật). Tự build calendar component.
