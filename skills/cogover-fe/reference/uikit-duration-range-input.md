## Cách sử dụng DurationRangeInput Component của ui-kit

Đường dẫn: `ui-kit/src/components/DurationRangeInput`

Props: `durationRule`, `value` ([string,string]), `defaultValue` ([string,string]), `onChange` (([string,string]) => void), `onInputChange` (ChangeEvent), `fullWidth`, `error`, `showBorderOnBlur`, `helperText`, `color`, `readOnly`, `className`. Kế thừa `TextFieldProps`.

Format duration: `"1h 30min"`, `"2d 4h"`, `"1y 6m"`. Đơn vị mặc định: `y`, `m`, `w`, `d`, `h`, `min`, `s`, `ms` (từ lớn đến nhỏ, cách nhau bởi dấu cách).

```tsx
// Controlled
const [durationRange, setDurationRange] = useState<[string, string]>(["1h", "2h"]);
<DurationRangeInput value={durationRange} onChange={setDurationRange} />

// Uncontrolled
<DurationRangeInput defaultValue={["1h", "2h"]} onChange={handleChange} />
// Không trộn lẫn: không dùng cả value và defaultValue cùng lúc

// onChange (blur) vs onInputChange (keystroke)
<DurationRangeInput
  onChange={(value) => setDurationRange(value)}          // [string, string] đã format
  onInputChange={(e) => showSuggestion(e.target.value)}  // keystroke thô, không dùng để lấy value cuối
/>

// Validation
<DurationRangeInput
  value={value} onChange={setValue}
  error={hasError}
  helperText={hasError ? "Khoảng thời gian không hợp lệ" : undefined}
/>

// readOnly (ẩn border) vs disabled (nền xám)
<DurationRangeInput value={["1h", "2h"]} readOnly />    // hiển thị dạng xem
<DurationRangeInput value={["1h", "2h"]} disabled={isLoading} />  // tạm không khả dụng

// Tuỳ chỉnh ký hiệu (chỉ khi cần khác mặc định)
<DurationRangeInput
  durationRule={{
    year: { symbol: "năm", equal: 12 }, month: { symbol: "tháng", equal: 4 },
    week: { symbol: "tuần", equal: 7 }, day: { symbol: "ngày", equal: 24 },
    hour: { symbol: "giờ", equal: 60 }, minute: { symbol: "phút", equal: 60 },
    second: { symbol: "giây", equal: 1000 }, millisecond: { symbol: "ms" },
  }}
  onChange={handleChange}
/>
```

**DO:** Dùng `value` là tuple `[string, string]`. Dùng `onChange` để nhận giá trị đã format (khi blur). Dùng `error`+`helperText` cho validation. Dùng `readOnly` thay `disabled` khi chỉ hiển thị. Bỏ `durationRule` nếu dùng ký hiệu mặc định.

**DON'T:** Truyền string đơn hoặc mảng 1 phần tử. Dùng `onInputChange` để lấy giá trị cuối. Trộn `value` và `defaultValue`. Render thông báo lỗi bên ngoài. Truyền lại `durationRule` mặc định.

| Đơn vị | Ký hiệu | Quy đổi |
|--------|---------|---------|
| year | `y` | 12m | month | `m` | 4w | week | `w` | 7d |
| day | `d` | 24h | hour | `h` | 60min | minute | `min` | 60s |
| second | `s` | 1000ms | millisecond | `ms` | — |
