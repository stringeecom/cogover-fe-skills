---
title: Cách sử dụng component Switch trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến trạng thái checked không đồng bộ, kích thước hiển thị sai, và lỗi accessibility
tags: switch, toggle, form, input, ui-kit
---

## Cách sử dụng component Switch trong ui-kit

Đường dẫn component: `ui-kit/src/components/Switch`

Props: `checked`, `onChange`, `disabled`, `size`, `className`. Extends `Omit<InputHTMLAttributes<HTMLInputElement>, "size" | "defaultChecked">`.

Giá trị mặc định: `size="medium"`, `checked` không truyền thì chạy uncontrolled mode.

---

### RULE-SWITCH-01: Pair `checked` với `onChange` trong controlled mode — omit `checked` cho uncontrolled

Khi truyền `checked`, component chạy controlled mode — state do bạn quản lý. Khi không truyền `checked`, component tự quản lý state nội bộ. Không truyền `checked` mà thiếu `onChange` (sẽ không cập nhật được).

**Sai:**

```tsx
// ❌ Truyền checked nhưng thiếu onChange — switch bị "đóng băng"
<Switch checked={isEnabled} />

// ❌ Dùng defaultChecked — prop này đã bị omit khỏi type
<Switch defaultChecked={true} onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅ Controlled mode
<Switch checked={isEnabled} onChange={(e) => setIsEnabled(e.target.checked)} />

// ✅ Uncontrolled mode — component tự quản lý state
<Switch onChange={(e) => handleChange(e)} />
```

---

### RULE-SWITCH-02: Sử dụng `size` cho kích thước — không override width/height qua className

Hai kích thước có sẵn: `"large"` (40x24px), `"medium"` (36x20px, mặc định). Không tự override kích thước qua className.

**Sai:**

```tsx
// ❌ Override kích thước qua className — phá vỡ tỷ lệ thumb và track
<Switch checked={value} onChange={handleChange} className={cx("w-[3rem] h-[1.5rem]")} />
```

**Đúng:**

```tsx
// ✅
<Switch checked={value} onChange={handleChange} size="large" />
<Switch checked={value} onChange={handleChange} size="medium" />
```

---

### RULE-SWITCH-03: Không truyền `type` — component luôn là checkbox

Input type luôn được set cứng là `"checkbox"` bên trong component. Không truyền `type` prop.

**Sai:**

```tsx
// ❌ Thừa — type luôn là "checkbox"
<Switch type="checkbox" checked={value} onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅
<Switch checked={value} onChange={handleChange} />
```

---

### RULE-SWITCH-04: Không tự build giao diện switch bằng div/span — sử dụng component Switch

Component Switch đã xử lý đầy đủ: track, thumb, animation transition, trạng thái disabled, focus-visible, và keyboard interaction (phím Space). Không tự build lại.

**Sai:**

```tsx
// ❌ Tự build switch — thiếu keyboard interaction và accessibility
<span
  className={cx("relative inline-block w-[2.25rem] h-[1.25rem] rounded-full", {
    "bg-primary-main": isOn,
    "bg-divider-primary": !isOn,
  })}
  onClick={() => setIsOn(!isOn)}
>
  <span
    className={cx("absolute top-[0.125rem] inline-block w-[1rem] h-[1rem] bg-[#fff] rounded-full transition-all", {
      "left-[0.125rem]": !isOn,
      "left-[1.125rem]": isOn,
    })}
  />
</span>
```

**Đúng:**

```tsx
// ✅ Component đã xử lý đầy đủ track, thumb, animation, keyboard, và focus-visible
<Switch checked={isOn} onChange={(e) => setIsOn(e.target.checked)} />
```

---

### RULE-SWITCH-05: Sử dụng `disabled` prop — không override style disabled qua className

Trạng thái disabled đã được xử lý nội bộ: thay đổi màu track và ẩn cursor pointer. Không tự override style disabled.

**Sai:**

```tsx
// ❌ Override style disabled qua className
<Switch
  checked={value}
  onChange={handleChange}
  className={cx("opacity-50 pointer-events-none")}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng disabled prop
<Switch checked={value} onChange={handleChange} disabled />

// ✅ Disabled dựa trên điều kiện
<Switch checked={value} onChange={handleChange} disabled={isLoading} />
```

---

### RULE-SWITCH-06: Không sử dụng `Checkbox` khi ngữ nghĩa là bật/tắt — dùng `Switch`

`Switch` dành cho hành động bật/tắt (toggle on/off) có hiệu lực ngay lập tức. `Checkbox` dành cho việc chọn/bỏ chọn trong form. Không dùng `Checkbox` khi ngữ nghĩa là bật/tắt.

**Sai:**

```tsx
// ❌ Dùng Checkbox cho toggle bật/tắt
<Checkbox
  checked={isDarkMode}
  onChange={(e) => setIsDarkMode(e.target.checked)}
/>
<span>Chế độ tối</span>
```

**Đúng:**

```tsx
// ✅ Switch cho hành động bật/tắt
<Switch checked={isDarkMode} onChange={(e) => setIsDarkMode(e.target.checked)} />
```

---

## Tham chiếu kích thước

| size | width | height | thumb |
|------|-------|--------|-------|
| `medium` | 36px | 20px | 16px |
| `large` | 40px | 24px | 20px |
