---
title: Cách sử dụng component CountrySelect trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến danh sách quốc gia sai, dial code/currency code không hiển thị, và state không đồng bộ
tags: country-select, select, country, flag, dial-code, currency, ui-kit
---

## Cách sử dụng component CountrySelect trong ui-kit

Đường dẫn component: `ui-kit/src/components/CountrySelect`

Props tùy chọn: `value`, `defaultValue`, `countries`, `onChange`, `disabled`, `showDialCode`, `showCurrencyCode`. Kế thừa các props từ `SelectProps` (ngoại trừ value, onChange, getValue, getLabel, options, groupOptions).

Giá trị mặc định: `countries=tất cả quốc gia`, `defaultValue=countries[0]`, `showDialCode=true`, `showCurrencyCode=false`, `showCheckIcon=false` (hardcoded), `searchable=false` (hardcoded).

Hardcoded: `optionListClassName="!w-[320px] !max-w-[90vw]"`, `className` bao gồm `[&_.stringee-text-field]:w-[90px]`.

---

### RULE-COUNTRY-SELECT-01: Sử dụng mã ISO2 cho `value` và `countries` — không dùng tên quốc gia

`value`, `defaultValue`, và mảng `countries` đều nhận mã ISO2 viết thường (2 ký tự). Không truyền tên đầy đủ hoặc mã ISO3.

**Sai:**

```tsx
// ❌ Tên quốc gia — không match được
<CountrySelect value="Vietnam" onChange={setCountry} />

// ❌ Mã ISO3 — không match được
<CountrySelect value="VNM" onChange={setCountry} />

// ❌ Viết hoa — phải viết thường
<CountrySelect value="VN" onChange={setCountry} />
```

**Đúng:**

```tsx
// ✅ Mã ISO2 viết thường
<CountrySelect value="vn" onChange={setCountry} />

// ✅ Lọc danh sách quốc gia
<CountrySelect
  countries={["vn", "us", "gb", "jp", "kr"]}
  value={country}
  onChange={setCountry}
/>
```

---

### RULE-COUNTRY-SELECT-02: Pair `value` với `onChange` cho controlled mode — omit `value` cho uncontrolled

Khi truyền `value`, component chạy controlled mode. Khi không truyền `value`, component dùng `defaultValue` và tự quản lý state. Không truyền `value` mà thiếu `onChange`.

**Sai:**

```tsx
// ❌ Truyền value nhưng thiếu onChange — select bị "đóng băng"
<CountrySelect value="vn" />
```

**Đúng:**

```tsx
// ✅ Controlled
<CountrySelect value={country} onChange={setCountry} />

// ✅ Uncontrolled với defaultValue
<CountrySelect defaultValue="vn" onChange={setCountry} />
```

---

### RULE-COUNTRY-SELECT-03: Sử dụng `showDialCode` và `showCurrencyCode` để hiển thị thông tin bổ sung

`showDialCode=true` (mặc định) hiển thị mã vùng (+84, +1...). `showCurrencyCode=false` (mặc định) ẩn mã tiền tệ. Thông tin bổ sung chỉ hiển thị trong dropdown options, không hiển thị trong giá trị đã chọn (chỉ hiện cờ).

**Sai:**

```tsx
// ❌ Tự build hiển thị dial code bên ngoài — duplicate với logic nội bộ
<div className={cx("flex items-center gap-[0.5rem]")}>
  <CountrySelect value={country} onChange={setCountry} />
  <span>+{getDialCode(country)}</span>
</div>
```

**Đúng:**

```tsx
// ✅ Hiện cả dial code và currency code
<CountrySelect
  value={country}
  onChange={setCountry}
  showDialCode
  showCurrencyCode
/>

// ✅ Chỉ hiện tên quốc gia, tắt dial code
<CountrySelect
  value={country}
  onChange={setCountry}
  showDialCode={false}
/>
```

---

### RULE-COUNTRY-SELECT-04: Sử dụng `countries` để lọc danh sách — không tự filter options

Truyền mảng ISO2 codes vào `countries` để chỉ hiển thị các quốc gia cần thiết. Component tự map và filter. Không tự build Select với country data.

**Sai:**

```tsx
// ❌ Tự build Select với country data — mất flag, dial code logic
<Select
  options={myCountryList}
  value={country}
  onChange={setCountry}
  getValue={(c) => c.code}
  getLabel={(c) => c.name}
/>
```

**Đúng:**

```tsx
// ✅ Chỉ hiện các quốc gia ASEAN
<CountrySelect
  countries={["vn", "th", "sg", "my", "id", "ph", "mm", "kh", "la", "bn"]}
  value={country}
  onChange={setCountry}
/>
```

---

### RULE-COUNTRY-SELECT-05: Không override các props hardcoded — `searchable`, `showCheckIcon`, `getValue`, `getLabel`, `options`

Các props này đã được set cứng bên trong component: `searchable=false`, `showCheckIcon=false`, `getValue`, `getLabel`, `options`. Không truyền lại chúng.

**Sai:**

```tsx
// ❌ Truyền các props đã hardcoded — bị ignore hoặc conflict
<CountrySelect
  value={country}
  onChange={setCountry}
  searchable
  showCheckIcon
  getValue={(c) => c.name}
/>
```

**Đúng:**

```tsx
// ✅ Chỉ truyền các props được hỗ trợ
<CountrySelect
  value={country}
  onChange={setCountry}
  showDialCode
  disabled={isLoading}
/>
```
