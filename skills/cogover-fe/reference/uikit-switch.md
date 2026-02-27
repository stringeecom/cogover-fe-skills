## Switch Component (`uikit-switch`)

Đường dẫn: `ui-kit/src/components/Switch`

Props: `checked`, `onChange`, `disabled`, `size` ("medium"|"large"), `className`. Extends `Omit<InputHTMLAttributes<HTMLInputElement>, "size" | "defaultChecked">`.

| size | width | height | thumb |
|------|-------|--------|-------|
| `medium` (mặc định) | 36px | 20px | 16px |
| `large` | 40px | 24px | 20px |

---

### RULE-SWITCH-01: Pair `checked` với `onChange` — omit `checked` cho uncontrolled

```tsx
// ❌ Controlled nhưng thiếu onChange — bị đóng băng
<Switch checked={isEnabled} />

// ❌ defaultChecked đã bị omit khỏi type
<Switch defaultChecked={true} onChange={handleChange} />

// ✅ Controlled
<Switch checked={isEnabled} onChange={(e) => setIsEnabled(e.target.checked)} />

// ✅ Uncontrolled
<Switch onChange={(e) => handleChange(e)} />
```

---

### RULE-SWITCH-02: Dùng prop `size` — không override width/height qua className

```tsx
// ❌ Phá vỡ tỷ lệ thumb và track
<Switch checked={value} onChange={handleChange} className={cx("w-[3rem] h-[1.5rem]")} />

// ✅
<Switch checked={value} onChange={handleChange} size="large" />
```

---

### RULE-SWITCH-03: Không tự build giao diện switch — dùng component Switch

Switch đã xử lý: track, thumb, animation, disabled, focus-visible, keyboard (Space).

```tsx
// ❌ Tự build — thiếu keyboard interaction và accessibility
<span className={cx("relative inline-block ...")} onClick={() => setIsOn(!isOn)}>
  <span className={cx("absolute ...")} />
</span>

// ✅
<Switch checked={isOn} onChange={(e) => setIsOn(e.target.checked)} />
```

---

### RULE-SWITCH-04: Dùng prop `disabled` — không override style qua className

```tsx
// ❌
<Switch checked={value} onChange={handleChange} className={cx("opacity-50 pointer-events-none")} />

// ✅
<Switch checked={value} onChange={handleChange} disabled={isLoading} />
```

---

### RULE-SWITCH-05: Dùng `Switch` cho bật/tắt — không dùng `Checkbox`

`Switch` = hành động bật/tắt có hiệu lực ngay. `Checkbox` = chọn/bỏ chọn trong form.

```tsx
// ❌
<Checkbox checked={isDarkMode} onChange={(e) => setIsDarkMode(e.target.checked)} />

// ✅
<Switch checked={isDarkMode} onChange={(e) => setIsDarkMode(e.target.checked)} />
```
