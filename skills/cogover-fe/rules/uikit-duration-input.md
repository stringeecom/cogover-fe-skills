---
title: Cách sử dụng DurationInput Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng DurationInput sai dẫn đến việc format thời lượng không đúng, xử lý onChange sai kiểu dữ liệu, và cấu hình durationRule không chính xác
tags: duration, input, time, durationRule, ui-kit
---

## Cách sử dụng DurationInput Component của ui-kit

Đường dẫn component: `ui-kit/src/components/DurationInput`

Props riêng: `durationRule`, `value`, `defaultValue`, `onChange`, `onInputChange`. Kế thừa tất cả props của `TextField` (trừ `onChange`, `value`, `defaultValue` đã được override) và hỗ trợ `ref` forwarding.

DurationInput là một input cho phép người dùng nhập giá trị thời lượng dưới dạng chuỗi có đơn vị (ví dụ: `1y 2m`, `3d 12h`, `30min 15s`). Component tự động validate và format giá trị khi blur.

---

### Đơn vị thời lượng mặc định

| Đơn vị | Symbol | Quy đổi sang đơn vị tiếp theo |
|--------|--------|-------------------------------|
| year | `y` | 12 (months) |
| month | `m` | 4 (weeks) |
| week | `w` | 7 (days) |
| day | `d` | 24 (hours) |
| hour | `h` | 60 (minutes) |
| minute | `min` | 60 (seconds) |
| second | `s` | 1000 (milliseconds) |
| millisecond | `ms` | — |

---

### RULE-DURATION-01: Sử dụng `onChange` để nhận giá trị đã format, dùng `onInputChange` để theo dõi giá trị đang nhập

`onChange` được gọi khi blur với giá trị đã được validate và format (kiểu `string`). `onInputChange` được gọi mỗi khi người dùng gõ phím (nhận `React.ChangeEvent<HTMLInputElement>`). Không nhầm lẫn giữa hai callback này.

**Sai:**

```tsx
// ❌ onChange nhận string, không phải event
<DurationInput
  onChange={(e) => {
    setValue(e.target.value);
  }}
/>
```

**Đúng:**

```tsx
// ✅ onChange nhận string đã format
<DurationInput
  onChange={(value) => {
    setValue(value);
  }}
/>

// ✅ Dùng onInputChange khi cần theo dõi giá trị đang gõ
<DurationInput
  onInputChange={(e) => {
    console.log("123:", e.target.value);
  }}
  onChange={(formattedValue) => {
    setValue(formattedValue);
  }}
/>
```

---

### RULE-DURATION-02: Truyền `value` và `defaultValue` dưới dạng chuỗi có đơn vị, không truyền số

Giá trị của DurationInput là chuỗi chứa số kèm đơn vị (ví dụ: `"1d 2h"`). Không truyền giá trị số hoặc chuỗi chỉ chứa số.

**Sai:**

```tsx
// ❌ Truyền số — component cần chuỗi có đơn vị
<DurationInput value={3600} />

// ❌ Chuỗi chỉ chứa số — không có đơn vị
<DurationInput value="3600" />
```

**Đúng:**

```tsx
// ✅ Chuỗi có đơn vị đúng format
<DurationInput value="1h 30min" />

// ✅ Sử dụng defaultValue cho giá trị ban đầu (uncontrolled)
<DurationInput defaultValue="2d 12h" />
```

---

### RULE-DURATION-03: Chỉ truyền `durationRule` khi cần tuỳ chỉnh đơn vị hoặc tỷ lệ quy đổi

Component đã có `defaultDurationRule` với đầy đủ các đơn vị từ year đến millisecond. Chỉ truyền `durationRule` khi cần thay đổi symbol hoặc tỷ lệ quy đổi.

**Sai:**

```tsx
// ❌ Truyền lại đúng giá trị mặc định — dư thừa
<DurationInput
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
<DurationInput onChange={handleChange} />

// ✅ Chỉ truyền khi cần tuỳ chỉnh (ví dụ: 1 tháng = 30 ngày thay vì 4 tuần)
<DurationInput
  durationRule={{
    year: { symbol: "y", equal: 12 },
    month: { symbol: "m", equal: 30 },
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

---

### RULE-DURATION-04: Các đơn vị trong giá trị phải theo đúng thứ tự từ lớn đến nhỏ

Khi nhập giá trị, các đơn vị phải theo thứ tự: `y` > `m` > `w` > `d` > `h` > `min` > `s` > `ms`. Giá trị không đúng thứ tự sẽ bị đánh dấu lỗi (error).

**Sai:**

```tsx
// ❌ Đơn vị không đúng thứ tự — sẽ bị validate lỗi
<DurationInput value="2h 1d" />

// ❌ min phải đứng sau h
<DurationInput value="30min 2h" />
```

**Đúng:**

```tsx
// ✅ Thứ tự từ lớn đến nhỏ
<DurationInput value="1d 2h" />

// ✅
<DurationInput value="2h 30min" />

// ✅ Có thể bỏ qua đơn vị ở giữa
<DurationInput value="1y 5d 30min" />
```

---

### RULE-DURATION-05: Sử dụng các props kế thừa từ TextField đúng cách

DurationInput kế thừa tất cả props của TextField. Sử dụng `label`, `helperText`, `error`, `placeholder`, `fullWidth`, v.v. như với TextField thông thường.

**Đúng:**

```tsx
// ✅ Sử dụng đầy đủ các props của TextField
<DurationInput
  label="Thời hạn"
  placeholder="Ví dụ: 1y 6m"
  helperText="Nhập thời lượng với các đơn vị: y, m, w, d, h, min, s"
  fullWidth
  onChange={handleChange}
/>

// ✅ Controlled mode với error handling bên ngoài
<DurationInput
  value={duration}
  onChange={setDuration}
  error={isError}
  helperText={isError ? "Giá trị không hợp lệ" : undefined}
/>
```

---

### RULE-DURATION-06: Giá trị tự động format khi blur

Component tự động chuẩn hoá giá trị khi người dùng rời khỏi input (blur). Ví dụ: nhập `25h` sẽ tự động chuyển thành `1d 1h`. Không cần tự format giá trị ở bên ngoài.

**Sai:**

```tsx
// ❌ Tự format bên ngoài — component đã xử lý
const handleChange = (value: string) => {
  const formatted = formatDuration(value);
  setDuration(formatted);
};

<DurationInput value={duration} onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅ Để component tự xử lý format
<DurationInput value={duration} onChange={setDuration} />
```
