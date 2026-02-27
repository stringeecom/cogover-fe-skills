## Cách sử dụng DurationInput Component của ui-kit

Đường dẫn: `ui-kit/src/components/DurationInput`

Input nhập giá trị thời lượng dạng chuỗi có đơn vị (ví dụ: `1y 2m`, `3d 12h`). Tự động validate và format khi blur. Kế thừa props của `TextField` (trừ `onChange`, `value`, `defaultValue`).

Props riêng: `durationRule`, `value`, `defaultValue`, `onChange`, `onInputChange`.

### Đơn vị mặc định

| Đơn vị | Symbol | Quy đổi |
|--------|--------|---------|
| year | `y` | 12 months |
| month | `m` | 4 weeks |
| week | `w` | 7 days |
| day | `d` | 24 hours |
| hour | `h` | 60 minutes |
| minute | `min` | 60 seconds |
| second | `s` | 1000 ms |
| millisecond | `ms` | — |

---

### RULE-DURATION-01: `onChange` nhận `string` (khi blur), `onInputChange` nhận `ChangeEvent` (khi gõ)

```tsx
// ❌ onChange nhận string, không phải event
<DurationInput onChange={(e) => setValue(e.target.value)} />

// ✅
<DurationInput
  onChange={(value) => setValue(value)}          // string đã format, khi blur
  onInputChange={(e) => console.log(e.target.value)}  // event khi gõ
/>
```

---

### RULE-DURATION-02: Truyền `value` dưới dạng chuỗi có đơn vị

```tsx
// ❌
<DurationInput value={3600} />
<DurationInput value="3600" />

// ✅
<DurationInput value="1h 30min" />
<DurationInput defaultValue="2d 12h" />
```

---

### RULE-DURATION-03: Các đơn vị phải theo thứ tự từ lớn đến nhỏ

```tsx
// ❌ Đơn vị sai thứ tự — bị validate lỗi
<DurationInput value="2h 1d" />

// ✅
<DurationInput value="1d 2h" />
<DurationInput value="1y 5d 30min" />  // Có thể bỏ đơn vị giữa
```

---

### RULE-DURATION-04: Giá trị tự động format khi blur — không tự format bên ngoài

```tsx
// ❌ Tự format bên ngoài — component đã xử lý (ví dụ: 25h → 1d 1h)
const handleChange = (value: string) => {
  const formatted = formatDuration(value);
  setDuration(formatted);
};

// ✅
<DurationInput value={duration} onChange={setDuration} />
```

---

### RULE-DURATION-05: Chỉ truyền `durationRule` khi cần tuỳ chỉnh

```tsx
// ✅ Bỏ qua khi dùng mặc định
<DurationInput onChange={handleChange} />

// ✅ Tùy chỉnh (ví dụ: 1 tháng = 30 ngày thay vì 4 tuần)
<DurationInput
  durationRule={{
    year: { symbol: "y", equal: 12 }, month: { symbol: "m", equal: 30 },
    week: { symbol: "w", equal: 7 }, day: { symbol: "d", equal: 24 },
    hour: { symbol: "h", equal: 60 }, minute: { symbol: "min", equal: 60 },
    second: { symbol: "s", equal: 1000 }, millisecond: { symbol: "ms" },
  }}
  onChange={handleChange}
/>
```
