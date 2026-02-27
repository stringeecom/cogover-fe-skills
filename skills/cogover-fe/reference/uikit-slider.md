---
title: Cách sử dụng Slider Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến uncontrolled/controlled mismatch, giá trị khởi tạo sai, hoặc conflicts về styling
tags: slider, range, input, ui-kit
---

## Cách sử dụng Slider Component của ui-kit

Đường dẫn component: `ui-kit/src/components/Slider`

Props: `value`, `min`, `max`, `step`, `onChange`, `className` (+ tất cả `InputHTMLAttributes<HTMLInputElement>` hợp lệ trừ `type`, `defaultValue`).

---

### RULE-SLIDER-01: Pair `value` với `onChange` trong controlled mode

Khi `value` được cung cấp dưới dạng number, component ở **controlled mode** — internal state bị bỏ qua và slider position được điều khiển hoàn toàn bởi external `value`. Bạn phải xử lý `onChange` để cập nhật value đó, nếu không thumb sẽ không di chuyển.

**Sai:**
```tsx
// ❌ Controlled value nhưng không có onChange — slider bị đóng băng
const [volume, setVolume] = useState(50);

<Slider value={volume} />
```

**Đúng:**
```tsx
// ✅ Luôn cập nhật external state qua onChange
const [volume, setVolume] = useState(50);

<Slider
  value={volume}
  onChange={(e) => setVolume(Number(e.target.value))}
/>
```

---

### RULE-SLIDER-02: Omit `value` cho uncontrolled usage; KHÔNG pass `defaultValue`

Khi bạn không cần track giá trị slider externally, omit `value` hoàn toàn — component tự quản lý internal state bắt đầu từ `0`. Prop `defaultValue` đã bị loại bỏ khỏi interface và sẽ không hoạt động.

**Sai:**
```tsx
// ❌ defaultValue không được hỗ trợ — không có hiệu lực
<Slider defaultValue={30} onChange={(e) => console.log("123:", e.target.value)} />
```

**Đúng:**
```tsx
// ✅ Uncontrolled — internal state bắt đầu từ 0
<Slider onChange={(e) => console.log("123:", e.target.value)} />

// ✅ Controlled với giá trị khởi tạo — sử dụng state thay thế
const [level, setLevel] = useState(30);
<Slider value={level} onChange={(e) => setLevel(Number(e.target.value))} />
```

---

### RULE-SLIDER-03: `className` style outer wrapper, không phải input

Component wrap một `<div>` xung quanh `<input type="range">`. Prop `className` được apply vào outer `<div>` đó. Input itself luôn sử dụng internal class `stringee-slider-input` và không thể override qua `className`.

**Sai:**
```tsx
// ❌ Mong đợi className thay đổi appearance của input trực tiếp
<Slider className="h-[0.25rem]" value={volume} onChange={handleChange} />
```

**Đúng:**
```tsx
// ✅ Sử dụng className để style container (width, margin, v.v.)
<Slider className="w-full my-[0.5rem]" value={volume} onChange={handleChange} />
```

---

### RULE-SLIDER-04: Sử dụng `min`, `max`, và `step` cho range control; không tính toán lại externally

`min` mặc định là `0`, `max` là `100`, `step` là `1`. Filled-track width được tính internally từ các giá trị này. Việc truyền chúng đảm bảo track render đúng tương ứng với thumb position.

**Sai:**
```tsx
// ❌ Thiếu min/max khiến filled track được tính dựa trên 0–100
//    ngay cả khi intended range là 0–10
<Slider value={value} onChange={(e) => setValue(Number(e.target.value))} />
// rồi tự clamp giá trị: setValue(Math.min(10, Number(e.target.value)))
```

**Đúng:**
```tsx
// ✅ Khai báo intended range để filled track khớp với thumb
<Slider
  value={value}
  min={0}
  max={10}
  step={0.5}
  onChange={(e) => setValue(Number(e.target.value))}
/>
```

---

### RULE-SLIDER-05: Không bao giờ pass `type` — nó luôn là `"range"`

`type` bị loại bỏ khỏi `SliderProps`. Component luôn render `<input type="range">`. Việc pass bất kỳ type nào khác sẽ không có hiệu lực và có thể gây TypeScript errors.

**Sai:**
```tsx
// ❌ type không có trong SliderProps — TypeScript error + không có hiệu lực
<Slider type="number" value={volume} onChange={handleChange} />
```

**Đúng:**
```tsx
// ✅ Đơn giản đừng pass type
<Slider value={volume} onChange={handleChange} />
```
