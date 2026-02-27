---
title: Cách sử dụng PhoneInput Component của ui-kit
impact: HIGH
impactDescription: Sử dụng PhoneInput sai dẫn đến việc nút gọi điện không hoạt động, quốc gia mặc định không chính xác, và duplicate logic với TextField + CountrySelect
tags: phone, input, call, country-select, text-field, ui-kit
---

## Cách sử dụng PhoneInput Component của ui-kit

Đường dẫn component: `ui-kit/src/components/PhoneInput`

PhoneInput là wrapper của `TextField` kết hợp với `CountrySelect` để nhập số điện thoại quốc tế. Props riêng: `value`, `onChange`, `defaultCountry`, `autoFillDialCode`, `countries`, `countrySelectProps`, `intlPhoneInputProps`, `isShowButtonCall`, `isShowButtonCallWhenHover`, `handleCall`. Các props còn lại kế thừa từ `TextFieldProps` (ngoại trừ `value` và `onChange` đã được override): `label`, `error`, `helperText`, `fullWidth`, `disabled`, `placeholder`, `className`, `inputClassName`, và tất cả `React.InputHTMLAttributes`. Hỗ trợ `ref` forwarding.

---

### RULE-PHONE-01: Sử dụng `PhoneInput` thay vì tự tạo TextField + CountrySelect

Khi cần input số điện thoại có chọn quốc gia, sử dụng `PhoneInput`. Không tự ghép `TextField` với `CountrySelect` thủ công.

**Sai:**

```tsx
// ❌ Tự tạo layout TextField + CountrySelect — PhoneInput đã làm điều này
<div className={cx("inline-flex")}>
  <CountrySelect value={countryCode} onChange={setCountryCode} />
  <TextField value={phone} onChange={handleChange} />
</div>
```

**Đúng:**

```tsx
// ✅ PhoneInput đã tích hợp sẵn CountrySelect và xử lý format số điện thoại
<PhoneInput value={phone} onChange={handleChange} />
```

---

### RULE-PHONE-02: Luôn truyền `value` và `onChange` để component hoạt động đúng

`onChange` trả về chuỗi số điện thoại đầy đủ (bao gồm mã quốc gia, ví dụ `+84123456789`). Nếu không truyền `value`, nút gọi điện sẽ không hoạt động và `autoFillDialCode` sẽ không có hiệu lực.

**Sai:**

```tsx
// ❌ Không truyền value — nút gọi điện không hoạt động
<PhoneInput onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅ Truyền cả value và onChange
<PhoneInput value={phone} onChange={handleChange} />
```

---

### RULE-PHONE-03: Sử dụng `isShowButtonCall` hoặc `isShowButtonCallWhenHover` cho nút gọi điện

`PhoneInput` hỗ trợ hiển thị nút gọi điện bên phải. Sử dụng `isShowButtonCall` để luôn hiển thị hoặc `isShowButtonCallWhenHover` để chỉ hiển thị khi hover. Không tự tạo nút gọi điện thủ công bên ngoài.

**Sai:**

```tsx
// ❌ Tự tạo nút gọi điện bên ngoài — PhoneInput đã hỗ trợ sẵn
<div className={cx("inline-flex")}>
  <PhoneInput value={phone} onChange={handleChange} />
  <button onClick={() => triggerStringeeXCall({ phoneNumber: phone })}>
    <FontAwesomeIcon icon={faPhone} />
  </button>
</div>
```

**Đúng:**

```tsx
// ✅ Luôn hiển thị nút gọi điện
<PhoneInput value={phone} onChange={handleChange} isShowButtonCall />

// ✅ Chỉ hiển thị nút gọi điện khi hover
<PhoneInput value={phone} onChange={handleChange} isShowButtonCallWhenHover />
```

---

### RULE-PHONE-04: Sử dụng `handleCall` khi cần tuỳ chỉnh hành vi gọi điện

Mặc định khi nhấn nút gọi, PhoneInput sẽ gọi `triggerStringeeXCall`. Nếu cần logic gọi khác, truyền `handleCall`.

**Đúng:**

```tsx
// ✅ Sử dụng hành vi gọi mặc định (triggerStringeeXCall)
<PhoneInput value={phone} onChange={handleChange} isShowButtonCall />

// ✅ Tuỳ chỉnh hành vi gọi
<PhoneInput
  value={phone}
  onChange={handleChange}
  isShowButtonCall
  handleCall={() => customCallFunction(phone)}
/>
```

---

### RULE-PHONE-05: Sử dụng `countries` để giới hạn danh sách quốc gia

Mặc định PhoneInput hiển thị tất cả quốc gia. Truyền `countries` với mảng mã ISO2 (viết thường) để giới hạn danh sách.

**Đúng:**

```tsx
// ✅ Chỉ hiển thị Việt Nam, Mỹ, Nhật Bản
<PhoneInput
  value={phone}
  onChange={handleChange}
  countries={["vn", "us", "jp"]}
/>
```

---

### RULE-PHONE-06: Sử dụng `fullWidth` khi cần PhoneInput chiếm toàn bộ chiều rộng

Mặc định PhoneInput là `inline-flex`. Truyền `fullWidth` để chiếm toàn bộ chiều rộng container cha.

**Sai:**

```tsx
// ❌ Dùng className để override chiều rộng
<PhoneInput className={cx("w-full")} value={phone} onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅ Sử dụng prop fullWidth
<PhoneInput fullWidth value={phone} onChange={handleChange} />
```

---

### RULE-PHONE-07: Không truyền `startAdornment` vì CountrySelect đã chiếm vị trí bên trái

PhoneInput sử dụng `startAdornment` nội bộ để hiển thị `CountrySelect`. Nếu truyền thêm `startAdornment`, nó sẽ bị ghi đè và CountrySelect sẽ không hiển thị.

**Sai:**

```tsx
// ❌ startAdornment ghi đè CountrySelect
<PhoneInput
  value={phone}
  onChange={handleChange}
  startAdornment={<FontAwesomeIcon icon={faPhone} />}
/>
```

**Đúng:**

```tsx
// ✅ Không truyền startAdornment — CountrySelect đã có sẵn
<PhoneInput value={phone} onChange={handleChange} />
```

---

### RULE-PHONE-08: Sử dụng `autoFillDialCode` để tự động điền mã quốc gia

Mặc định `autoFillDialCode` là `true`. Khi người dùng nhập số bắt đầu bằng `+0`, PhoneInput sẽ tự động thay thế bằng mã quốc gia mặc định của workspace khi blur. Truyền `autoFillDialCode={false}` nếu không muốn hành vi này.

**Đúng:**

```tsx
// ✅ Mặc định: tự động điền mã quốc gia
<PhoneInput value={phone} onChange={handleChange} />

// ✅ Tắt tự động điền mã quốc gia
<PhoneInput value={phone} onChange={handleChange} autoFillDialCode={false} />
```

---

### RULE-PHONE-09: Sử dụng các prop của TextField cho validation và trạng thái

PhoneInput kế thừa các props từ TextField (trừ `value` và `onChange`). Sử dụng `error`, `helperText`, `label`, `disabled`, `required` như khi dùng TextField thông thường.

**Đúng:**

```tsx
// ✅ Sử dụng error + helperText cho validation
<PhoneInput
  fullWidth
  label="Số điện thoại"
  required
  value={phone}
  onChange={handleChange}
  error={!isValidPhone}
  helperText={!isValidPhone ? "Số điện thoại không hợp lệ" : undefined}
/>

// ✅ Sử dụng disabled
<PhoneInput fullWidth value={phone} disabled />
```

---

### RULE-PHONE-10: Sử dụng helper `validatePhoneNumber` để kiểm tra số điện thoại

PhoneInput export sẵn hàm `validatePhoneNumber` để kiểm tra tính hợp lệ của số điện thoại. Không tự viết regex validation.

**Sai:**

```tsx
// ❌ Tự viết regex validation
const isValid = /^\+?[0-9]{10,15}$/.test(phone);
```

**Đúng:**

```tsx
// ✅ Sử dụng helper có sẵn
import { validatePhoneNumber } from "ui-kit/src/components/PhoneInput/helper";

const isValid = validatePhoneNumber(phone);

// ✅ Giới hạn validation theo danh sách quốc gia
const isValid = validatePhoneNumber(phone, ["vn", "us", "jp"]);
```

---

## Tham chiếu cấu trúc

| Phần tử | Mô tả |
|---------|-------|
| Container ngoài | `inline-flex`, thêm `w-full` khi `fullWidth` |
| CountrySelect (bên trái) | Nằm trong `startAdornment` của TextField, hiển thị cờ quốc gia |
| TextField (giữa) | Chiếm `flex-1`, border-radius phải bị loại bỏ khi có nút gọi |
| Nút gọi điện (bên phải) | `40x40px`, `rounded-r`, màu nền `bg-primary-light-90`, chỉ hiển thị khi `isShowButtonCall` hoặc `isShowButtonCallWhenHover` |

| Props riêng | Kiểu | Mặc định | Mô tả |
|-------------|------|----------|-------|
| `value` | `string` | — | Giá trị số điện thoại (bao gồm mã quốc gia) |
| `onChange` | `(value: string) => void` | — | Callback khi giá trị thay đổi |
| `defaultCountry` | `string` | Theo cấu hình workspace | Mã quốc gia ISO2 mặc định |
| `autoFillDialCode` | `boolean` | `true` | Tự động thay `+0` bằng mã quốc gia khi blur |
| `countries` | `string[]` | Tất cả quốc gia | Mảng mã ISO2 để giới hạn danh sách |
| `countrySelectProps` | `Partial<CountrySelectProps>` | — | Props tuỳ chỉnh cho CountrySelect |
| `intlPhoneInputProps` | `Partial<UsePhoneInputConfig>` | — | Props tuỳ chỉnh cho react-international-phone |
| `isShowButtonCall` | `boolean` | — | Luôn hiển thị nút gọi điện |
| `isShowButtonCallWhenHover` | `boolean` | — | Hiển thị nút gọi điện khi hover |
| `handleCall` | `() => void` | — | Tuỳ chỉnh hành vi khi nhấn nút gọi |
