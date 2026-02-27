## Cách sử dụng PhoneInput Component của ui-kit

Đường dẫn: `ui-kit/src/components/PhoneInput`

| Props riêng | Kiểu | Mặc định | Mô tả |
|-------------|------|----------|-------|
| `value` | `string` | — | Số điện thoại kể cả mã quốc gia |
| `onChange` | `(value: string) => void` | — | Trả về chuỗi đầy đủ (e.g. `+84123456789`) |
| `defaultCountry` | `string` | Cấu hình workspace | Mã ISO2 mặc định |
| `autoFillDialCode` | `boolean` | `true` | Tự động thay `+0` bằng mã quốc gia khi blur |
| `countries` | `string[]` | Tất cả | Mảng mã ISO2 để giới hạn danh sách |
| `countrySelectProps` | `Partial<CountrySelectProps>` | — | Props cho CountrySelect |
| `intlPhoneInputProps` | `Partial<UsePhoneInputConfig>` | — | Props cho react-international-phone |
| `isShowButtonCall` | `boolean` | — | Luôn hiển thị nút gọi điện |
| `isShowButtonCallWhenHover` | `boolean` | — | Hiển thị nút gọi điện khi hover |
| `handleCall` | `() => void` | — | Tuỳ chỉnh hành vi gọi (mặc định: triggerStringeeXCall) |

Kế thừa `TextFieldProps` (trừ `value`, `onChange`): `label`, `error`, `helperText`, `fullWidth`, `disabled`, `placeholder`, `className`, `inputClassName`, v.v.

```tsx
// Cơ bản
<PhoneInput value={phone} onChange={handleChange} />

// Với validation
<PhoneInput
  fullWidth
  label="Số điện thoại"
  required
  value={phone}
  onChange={handleChange}
  error={!isValidPhone}
  helperText={!isValidPhone ? "Số điện thoại không hợp lệ" : undefined}
/>

// Nút gọi điện
<PhoneInput value={phone} onChange={handleChange} isShowButtonCall />
<PhoneInput value={phone} onChange={handleChange} isShowButtonCallWhenHover handleCall={() => customCall(phone)} />

// Giới hạn quốc gia
<PhoneInput value={phone} onChange={handleChange} countries={["vn", "us", "jp"]} />
```

```ts
// Validate số điện thoại
import { validatePhoneNumber } from "ui-kit/src/components/PhoneInput/helper";
const isValid = validatePhoneNumber(phone);               // tất cả quốc gia
const isValid = validatePhoneNumber(phone, ["vn", "us"]); // giới hạn quốc gia
```

**DO:** Dùng `PhoneInput` thay vì tự ghép `TextField` + `CountrySelect`. Dùng `fullWidth` thay vì `className="w-full"`. Dùng `validatePhoneNumber` helper thay vì tự viết regex. Dùng `isShowButtonCall`/`isShowButtonCallWhenHover` cho nút gọi điện.

**DON'T:** Bỏ `value` (nút gọi điện không hoạt động). Tự tạo nút gọi điện bên ngoài. Truyền `startAdornment` (sẽ ghi đè CountrySelect).
