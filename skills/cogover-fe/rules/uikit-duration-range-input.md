---
title: Cách sử dụng DurationRangeInput Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng DurationRangeInput sai dẫn đến việc format duration không đúng, xử lý value range sai kiểu dữ liệu và nhầm lẫn giữa onChange và onInputChange
tags: duration, range, input, time, ui-kit
---

## Cách sử dụng DurationRangeInput Component của ui-kit

Đường dẫn component: `ui-kit/src/components/DurationRangeInput`

Props: `durationRule`, `value`, `defaultValue`, `onChange`, `onInputChange`, `fullWidth`, `error`, `showBorderOnBlur`, `helperText`, `color`, `readOnly`, `className`. Kế thừa các props từ `TextFieldProps` (ngoại trừ `onChange`, `value`, `defaultValue`).

Component này cho phép nhập khoảng thời gian dạng range (từ - đến) với format duration string. Ví dụ: `["1h 30min", "2h 45min"]`.

### Format duration string

Duration string sử dụng các ký hiệu đơn vị mặc định: `y` (year), `m` (month), `w` (week), `d` (day), `h` (hour), `min` (minute), `s` (second), `ms` (millisecond). Các đơn vị phải được sắp xếp từ lớn đến nhỏ và cách nhau bởi dấu cách.

Ví dụ hợp lệ: `"1h 30min"`, `"2d 4h"`, `"1y 6m"`, `"45s 200ms"`.

---

### RULE-DRI-01: Giá trị `value` và `defaultValue` phải là tuple `[string, string]`

`value` và `defaultValue` là tuple gồm 2 phần tử string, phần tử đầu là giá trị "từ", phần tử sau là giá trị "đến". Không truyền mảng có độ dài khác 2.

**Sai:**

```tsx
// ❌ Truyền chuỗi đơn — kiểu dữ liệu sai
<DurationRangeInput value="1h 30min" onChange={handleChange} />

// ❌ Truyền mảng 1 phần tử
<DurationRangeInput value={["1h 30min"]} onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅ Truyền tuple [string, string]
<DurationRangeInput value={["1h 30min", "2h 45min"]} onChange={handleChange} />

// ✅ Giá trị rỗng cho cả hai đầu
<DurationRangeInput value={["", ""]} onChange={handleChange} />

// ✅ Chỉ có giá trị một đầu
<DurationRangeInput value={["1h", ""]} onChange={handleChange} />
```

---

### RULE-DRI-02: Sử dụng `onChange` để nhận giá trị đã format, `onInputChange` để nhận sự kiện input thô

`onChange` được gọi khi blur với giá trị đã được validate và format thành `[string, string]`. `onInputChange` được gọi mỗi khi người dùng gõ ký tự, nhận `React.ChangeEvent<HTMLInputElement>` gốc. Không dùng `onInputChange` để lấy giá trị duration cuối cùng.

**Sai:**

```tsx
// ❌ Dùng onInputChange để lấy giá trị cuối cùng — giá trị chưa được format
<DurationRangeInput
  onInputChange={(e) => setDuration(e.target.value)}
/>
```

**Đúng:**

```tsx
// ✅ Dùng onChange để nhận giá trị đã format
<DurationRangeInput
  onChange={(value) => {
    // value là [string, string], ví dụ: ["1h 30min", "2h"]
    setDurationRange(value);
  }}
/>

// ✅ Dùng onInputChange khi cần theo dõi từng keystroke (ví dụ: hiển thị gợi ý)
<DurationRangeInput
  onChange={setDurationRange}
  onInputChange={(e) => {
    showSuggestion(e.target.value);
  }}
/>
```

---

### RULE-DRI-03: Tuỳ chỉnh `durationRule` khi cần thay đổi ký hiệu hoặc quy đổi đơn vị

Mặc định, component sử dụng `defaultDurationRule` với các ký hiệu chuẩn (`y`, `m`, `w`, `d`, `h`, `min`, `s`, `ms`). Chỉ truyền `durationRule` khi cần thay đổi ký hiệu hoặc tỷ lệ quy đổi. Không truyền lại giá trị mặc định.

**Sai:**

```tsx
// ❌ Truyền lại giá trị mặc định — dư thừa
<DurationRangeInput
  durationRule={{
    year: { symbol: "y", equal: 12 },
    month: { symbol: "m", equal: 4 },
    week: { symbol: "w", equal: 7 },
    day: { symbol: "d", equal: 24 },
    hour: { symbol: "h", equal: 60 },
    minute: { symbol: "min", equal: 60 },
    second: { symbol: "s", equal: 1000 },
    millisecond: { symbol: "ms" },
  }}
  onChange={handleChange}
/>
```

**Đúng:**

```tsx
// ✅ Bỏ qua khi dùng giá trị mặc định
<DurationRangeInput onChange={handleChange} />

// ✅ Chỉ truyền khi cần tuỳ chỉnh ký hiệu tiếng Việt
<DurationRangeInput
  durationRule={{
    year: { symbol: "năm", equal: 12 },
    month: { symbol: "tháng", equal: 4 },
    week: { symbol: "tuần", equal: 7 },
    day: { symbol: "ngày", equal: 24 },
    hour: { symbol: "giờ", equal: 60 },
    minute: { symbol: "phút", equal: 60 },
    second: { symbol: "giây", equal: 1000 },
    millisecond: { symbol: "ms" },
  }}
  onChange={handleChange}
/>
```

---

### RULE-DRI-04: Sử dụng controlled hoặc uncontrolled, không trộn lẫn

Khi dùng `value` (controlled), component sẽ ưu tiên giá trị từ props. Khi dùng `defaultValue` (uncontrolled), component tự quản lý state nội bộ. Không truyền cả `value` và `defaultValue` cùng lúc.

**Sai:**

```tsx
// ❌ Trộn lẫn controlled và uncontrolled
<DurationRangeInput
  value={durationRange}
  defaultValue={["1h", "2h"]}
  onChange={setDurationRange}
/>
```

**Đúng:**

```tsx
// ✅ Controlled — quản lý state bên ngoài
const [durationRange, setDurationRange] = useState<[string, string]>(["1h", "2h"]);
<DurationRangeInput value={durationRange} onChange={setDurationRange} />

// ✅ Uncontrolled — chỉ cần giá trị khởi tạo
<DurationRangeInput defaultValue={["1h", "2h"]} onChange={handleChange} />
```

---

### RULE-DRI-05: Sử dụng `error` và `helperText` để hiển thị lỗi validation

Truyền `error={true}` để hiển thị border lỗi và kết hợp với `helperText` để hiển thị thông báo lỗi bên dưới input. Không tự render thông báo lỗi bên ngoài component.

**Sai:**

```tsx
// ❌ Tự render thông báo lỗi bên ngoài
<DurationRangeInput value={value} onChange={setValue} />
{hasError && <span className={cx("text-error")}>Khoảng thời gian không hợp lệ</span>}
```

**Đúng:**

```tsx
// ✅ Sử dụng error và helperText tích hợp sẵn
<DurationRangeInput
  value={value}
  onChange={setValue}
  error={hasError}
  helperText={hasError ? "Khoảng thời gian không hợp lệ" : undefined}
/>
```

---

### RULE-DRI-06: Sử dụng `readOnly` thay vì `disabled` khi chỉ cần hiển thị giá trị

`readOnly` ẩn border và bỏ padding trái, phù hợp để hiển thị giá trị dạng xem. `disabled` thêm background xám và ngăn tương tác, phù hợp khi input tạm thời không khả dụng.

**Sai:**

```tsx
// ❌ Dùng disabled để hiển thị giá trị dạng xem — giao diện bị xám
<DurationRangeInput value={["1h", "2h"]} disabled />
```

**Đúng:**

```tsx
// ✅ Dùng readOnly để hiển thị giá trị dạng xem
<DurationRangeInput value={["1h", "2h"]} readOnly />

// ✅ Dùng disabled khi input tạm thời không khả dụng
<DurationRangeInput value={["1h", "2h"]} disabled={isLoading} />
```

---

## Tham chiếu style

| Trạng thái | Border | Background |
|------------|--------|------------|
| Mặc định | `border-divider-primary` | transparent |
| Hover | `border-primary-light-70` | transparent |
| Focus | `border-info` | transparent |
| Lỗi | `border-error` | transparent |
| Disabled | `border-primary-light-94` | `bg-primary-light-94` |
| `showBorderOnBlur={false}` | transparent | `hover:bg-primary-light-96` |
| `readOnly` | transparent | transparent |

## Tham chiếu đơn vị mặc định

| Đơn vị | Ký hiệu | Quy đổi |
|--------|----------|---------|
| year | `y` | 12 months |
| month | `m` | 4 weeks |
| week | `w` | 7 days |
| day | `d` | 24 hours |
| hour | `h` | 60 minutes |
| minute | `min` | 60 seconds |
| second | `s` | 1000 milliseconds |
| millisecond | `ms` | — |
