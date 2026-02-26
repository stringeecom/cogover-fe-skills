---
title: Cách sử dụng InputNumber Component của ui-kit
impact: HIGH
impactDescription: Sử dụng InputNumber sai dẫn đến việc format số không đúng, mất kiểm soát giá trị min/max, và hiển thị số thập phân sai
tags: input-number, number, format, min, max, step, decimal, ui-kit
---

## Cách sử dụng InputNumber Component của ui-kit

Đường dẫn component: `ui-kit/src/components/InputNumber`

Props riêng: `format`, `min`, `max`, `step`, `value`, `defaultValue`, `onChange`, `fillMissingZero`, `numberOfIntegerDigit`, `numberOfDecimalDigit`, `showIncreaseDecreaseButton`, `roundRule`, `precision`, `allowTypeOutsideRange`. Kế thừa tất cả props từ `TextField` (trừ `value`, `defaultValue`, `onChange` đã được override) và hỗ trợ `ref` forwarding.

---

### RULE-INPUTNUMBER-01: Sử dụng `format` phù hợp với yêu cầu hiển thị số

InputNumber hỗ trợ nhiều format hiển thị số. Format mặc định là `"###0"` (không có dấu phân cách). Chỉ truyền `format` khi cần thay đổi cách hiển thị.

**Các format số nguyên được hỗ trợ:**

| Format | Mô tả | Ví dụ |
|--------|--------|-------|
| `###0` | Không dấu phân cách (mặc định) | `1234567` |
| `# ##0` | Dấu cách phân cách hàng nghìn | `1 234 567` |
| `#,##0` | Dấu phẩy phân cách hàng nghìn (kiểu US) | `1,234,567` |
| `#.##0` | Dấu chấm phân cách hàng nghìn (kiểu DE) | `1.234.567` |
| `#,##,##0` | Dấu phẩy phân cách kiểu Ấn Độ | `12,34,567` |

**Các format số thập phân được hỗ trợ (thêm phần thập phân sau format số nguyên):**

| Format | Mô tả | Ví dụ |
|--------|--------|-------|
| `###0.00` | Không phân cách, 2 chữ số thập phân (dấu chấm) | `1234567.89` |
| `# ##0.00` | Dấu cách phân cách, thập phân dấu chấm | `1 234 567.89` |
| `#,##0.00` | Phẩy phân cách nghìn, thập phân dấu chấm | `1,234,567.89` |
| `###0,00` | Không phân cách, thập phân dấu phẩy | `1234567,89` |
| `# ##0,00` | Dấu cách phân cách, thập phân dấu phẩy | `1 234 567,89` |
| `#.##0,00` | Chấm phân cách nghìn, thập phân dấu phẩy (kiểu DE) | `1.234.567,89` |

**Sai:**

```tsx
// ❌ Dùng format số nguyên cho trường cần nhập số thập phân
<InputNumber format="###0" />

// ❌ Tự format bằng cách thêm logic bên ngoài
<InputNumber
  format="###0"
  onChange={(val) => setPrice(val ? parseFloat(val.toFixed(2)) : null)}
/>
```

**Đúng:**

```tsx
// ✅ Dùng format số thập phân phù hợp
<InputNumber format="#,##0.00" />

// ✅ Dùng format mặc định cho số nguyên đơn giản
<InputNumber />
```

---

### RULE-INPUTNUMBER-02: Sử dụng `onChange` với signature `(value: number | null) => void`

`onChange` của InputNumber trả về giá trị `number | null`, KHÔNG phải event object như input thông thường. Giá trị `null` khi input rỗng.

**Sai:**

```tsx
// ❌ Xử lý onChange như input thông thường
<InputNumber
  onChange={(e) => setQuantity(Number(e.target.value))}
/>

// ❌ Không xử lý trường hợp null
<InputNumber
  onChange={(value) => setQuantity(value + 1)}
/>
```

**Đúng:**

```tsx
// ✅ onChange nhận trực tiếp giá trị number | null
<InputNumber
  value={quantity}
  onChange={(value) => setQuantity(value)}
/>

// ✅ Xử lý cả trường hợp null
<InputNumber
  value={price}
  onChange={(value) => {
    if (value !== null) {
      setPrice(value);
    }
  }}
/>
```

---

### RULE-INPUTNUMBER-03: Sử dụng `min`/`max` kết hợp với `allowTypeOutsideRange` để kiểm soát giới hạn

Mặc định `allowTypeOutsideRange = true`, nghĩa là người dùng có thể nhập giá trị ngoài phạm vi `min`/`max` trong lúc đang gõ. Giá trị chỉ bị giới hạn khi nhấn nút tăng/giảm. Đặt `allowTypeOutsideRange={false}` để tự động clamp giá trị khi blur.

**Sai:**

```tsx
// ❌ Tự clamp giá trị bên ngoài component
<InputNumber
  min={0}
  max={100}
  onChange={(value) => {
    if (value !== null) {
      setAmount(Math.min(100, Math.max(0, value)));
    }
  }}
/>
```

**Đúng:**

```tsx
// ✅ Để component tự xử lý clamp khi blur
<InputNumber
  min={0}
  max={100}
  allowTypeOutsideRange={false}
  value={amount}
  onChange={setAmount}
/>

// ✅ Cho phép nhập ngoài phạm vi (mặc định), chỉ giới hạn nút tăng/giảm
<InputNumber
  min={0}
  max={100}
  value={amount}
  onChange={setAmount}
/>
```

---

### RULE-INPUTNUMBER-04: Sử dụng `showIncreaseDecreaseButton={false}` khi không cần nút tăng/giảm

Mặc định `showIncreaseDecreaseButton = true`, nút tăng/giảm sẽ luôn hiển thị. Khi không cần nút này, truyền `showIncreaseDecreaseButton={false}`. Khi ẩn nút tăng/giảm, `endAdornment` từ props sẽ được sử dụng thay thế.

**Sai:**

```tsx
// ❌ Tự ẩn nút bằng CSS
<InputNumber
  className="[&_button]:hidden"
  endAdornment={<span>VND</span>}
/>
```

**Đúng:**

```tsx
// ✅ Dùng prop để ẩn nút tăng/giảm, endAdornment sẽ hoạt động bình thường
<InputNumber
  showIncreaseDecreaseButton={false}
  endAdornment={<span>VND</span>}
/>

// ✅ Hiển thị nút tăng/giảm (mặc định, không cần truyền)
<InputNumber value={quantity} onChange={setQuantity} />
```

---

### RULE-INPUTNUMBER-05: Sử dụng `fillMissingZero` để tự động điền số 0 vào phần thập phân

Khi cần hiển thị đủ số chữ số thập phân (ví dụ: `1.50` thay vì `1.5`), bật `fillMissingZero={true}`. Mặc định là `false`.

**Sai:**

```tsx
// ❌ Tự format giá trị bên ngoài để thêm số 0
<InputNumber
  format="#,##0.00"
  onChange={(value) => {
    setPrice(value);
    setDisplayPrice(value?.toFixed(2));
  }}
/>
```

**Đúng:**

```tsx
// ✅ Dùng fillMissingZero để component tự điền đủ số 0
<InputNumber
  format="#,##0.00"
  fillMissingZero
  value={price}
  onChange={setPrice}
/>
// Khi nhập 1.5, sau khi blur sẽ hiển thị "1.50"
```

---

### RULE-INPUTNUMBER-06: Sử dụng `roundRule` và `precision` để làm tròn số

Khi cần làm tròn giá trị, sử dụng `roundRule` kết hợp với `precision`. `roundRule` hỗ trợ 3 giá trị: `"round"` (làm tròn thông thường), `"round_up"` (làm tròn lên), `"round_down"` (làm tròn xuống). `precision` mặc định là `2`.

**Sai:**

```tsx
// ❌ Tự làm tròn giá trị bên ngoài
<InputNumber
  format="#,##0.00"
  onChange={(value) => {
    setPrice(value !== null ? Math.round(value * 100) / 100 : null);
  }}
/>
```

**Đúng:**

```tsx
// ✅ Dùng roundRule và precision
<InputNumber
  format="#,##0.00"
  roundRule="round"
  precision={2}
  value={price}
  onChange={setPrice}
/>

// ✅ Làm tròn xuống với 3 chữ số thập phân
<InputNumber
  format="#,##0.000"
  roundRule="round_down"
  precision={3}
  value={rate}
  onChange={setRate}
/>
```

---

### RULE-INPUTNUMBER-07: Sử dụng `numberOfIntegerDigit` và `numberOfDecimalDigit` để giới hạn số chữ số

Mặc định cả `numberOfIntegerDigit` và `numberOfDecimalDigit` đều là `15`. Khi cần giới hạn số chữ số nhập, hãy truyền giá trị phù hợp. Lưu ý: giới hạn này chỉ hoạt động khi `allowTypeOutsideRange={false}`.

**Sai:**

```tsx
// ❌ Tự validate số chữ số bên ngoài
<InputNumber
  onChange={(value) => {
    if (value !== null && value.toString().length <= 10) {
      setAmount(value);
    }
  }}
/>
```

**Đúng:**

```tsx
// ✅ Dùng numberOfIntegerDigit và numberOfDecimalDigit
<InputNumber
  numberOfIntegerDigit={10}
  numberOfDecimalDigit={2}
  allowTypeOutsideRange={false}
  format="#,##0.00"
  value={amount}
  onChange={setAmount}
/>
```

---

### RULE-INPUTNUMBER-08: Không truyền rõ ràng giá trị mặc định

Các giá trị mặc định: `format="###0"`, `step={1}`, `showIncreaseDecreaseButton={true}`, `allowTypeOutsideRange={true}`, `numberOfIntegerDigit={15}`, `numberOfDecimalDigit={15}`, `precision={2}`. Chỉ truyền khi cần thay đổi.

**Sai:**

```tsx
// ❌ Truyền thừa giá trị mặc định
<InputNumber
  format="###0"
  step={1}
  showIncreaseDecreaseButton={true}
  allowTypeOutsideRange={true}
  numberOfIntegerDigit={15}
  numberOfDecimalDigit={15}
  precision={2}
  value={quantity}
  onChange={setQuantity}
/>
```

**Đúng:**

```tsx
// ✅ Chỉ truyền những gì cần thay đổi
<InputNumber value={quantity} onChange={setQuantity} />
```

---

## Tham chiếu props

| Prop | Kiểu | Mặc định | Mô tả |
|------|------|----------|--------|
| `format` | `string` | `"###0"` | Chuỗi format hiển thị số |
| `min` | `number` | `undefined` | Giá trị tối thiểu |
| `max` | `number` | `undefined` | Giá trị tối đa |
| `step` | `number` | `1` | Bước tăng/giảm khi nhấn nút |
| `value` | `number \| null` | `undefined` | Giá trị controlled |
| `defaultValue` | `number \| null` | `undefined` | Giá trị mặc định (uncontrolled) |
| `onChange` | `(value: number \| null) => void` | `undefined` | Callback khi giá trị thay đổi |
| `fillMissingZero` | `boolean` | `false` | Tự động điền số 0 vào phần thập phân |
| `numberOfIntegerDigit` | `number` | `15` | Số chữ số phần nguyên tối đa |
| `numberOfDecimalDigit` | `number` | `15` | Số chữ số phần thập phân tối đa |
| `showIncreaseDecreaseButton` | `boolean` | `true` | Hiển thị nút tăng/giảm |
| `roundRule` | `"round" \| "round_up" \| "round_down" \| null` | `undefined` | Quy tắc làm tròn |
| `precision` | `number` | `2` | Số chữ số thập phân khi làm tròn |
| `allowTypeOutsideRange` | `boolean` | `true` | Cho phép nhập giá trị ngoài phạm vi min/max |

Kế thừa từ `TextField`: `label`, `error`, `helperText`, `fullWidth`, `startAdornment`, `endAdornment`, `disabled`, `placeholder`, `className`, `inputClassName`, `readOnly`, `autoFocus`, và tất cả `React.InputHTMLAttributes<HTMLInputElement>`.
