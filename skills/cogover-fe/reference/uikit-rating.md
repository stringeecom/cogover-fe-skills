---
title: Cách sử dụng Rating Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến uncontrolled/controlled mismatch, icon không hiển thị đúng, hoặc không thể tương tác
tags: rating, star, icon, emoji, ui-kit
---

## Cách sử dụng Rating Component của ui-kit

Đường dẫn component: `ui-kit/src/components/Rating`

Props: `value`, `defaultValue`, `onChange`, `max`, `icon`, `iconSize`, `gap`, `direction`, `disabled`, `tabIndex` (+ tất cả `React.HTMLAttributes<HTMLDivElement>` hợp lệ trừ `onChange`, `defaultValue`).

---

### RULE-RATING-01: Pair `value` với `onChange` trong controlled mode

Khi `value` được cung cấp, component ở **controlled mode** — internal state bị bỏ qua và rating được điều khiển hoàn toàn bởi external `value`. Bạn phải xử lý `onChange` để cập nhật value đó, nếu không rating sẽ không thay đổi khi click.

**Sai:**
```tsx
// ❌ Controlled value nhưng không có onChange — rating bị đóng băng
const [score, setScore] = useState(3);

<Rating value={score} />
```

**Đúng:**
```tsx
// ✅ Luôn cập nhật external state qua onChange
const [score, setScore] = useState(3);

<Rating value={score} onChange={(val) => setScore(val)} />
```

---

### RULE-RATING-02: Omit `value` và `onChange` cho uncontrolled usage; dùng `defaultValue` để set giá trị khởi tạo

Khi bạn không cần track giá trị rating externally, omit `value` và `onChange` hoàn toàn — component tự quản lý internal state. Sử dụng `defaultValue` để set giá trị ban đầu.

**Sai:**
```tsx
// ❌ Truyền value mà không có onChange — không thể tương tác
<Rating value={3} />
```

**Đúng:**
```tsx
// ✅ Uncontrolled với giá trị khởi tạo
<Rating defaultValue={3} />

// ✅ Uncontrolled — internal state bắt đầu từ null
<Rating />
```

---

### RULE-RATING-03: `onChange` trả về `number | null`, không phải event

Callback `onChange` nhận trực tiếp giá trị `number | null`, không phải `ChangeEvent` như input thông thường. Khi click vào rating đã chọn, giá trị trả về là `0` (bỏ chọn).

**Sai:**
```tsx
// ❌ Mong đợi event object như input — sẽ lỗi runtime
<Rating value={score} onChange={(e) => setScore(Number(e.target.value))} />
```

**Đúng:**
```tsx
// ✅ onChange nhận trực tiếp giá trị number | null
<Rating value={score} onChange={(val) => setScore(val)} />
```

---

### RULE-RATING-04: Sử dụng prop `icon` để tuỳ chỉnh icon; không thay đổi children

Component không nhận `children` làm icon. Sử dụng prop `icon` để truyền URL ảnh hoặc emoji string. Khi không truyền `icon`, component tự động hiển thị ngôi sao mặc định.

**Sai:**
```tsx
// ❌ Truyền children — sẽ không hiển thị icon custom
<Rating>
  <HeartIcon />
</Rating>
```

**Đúng:**
```tsx
// ✅ Mặc định — hiển thị ngôi sao
<Rating value={score} onChange={setScore} />

// ✅ Custom icon bằng emoji
<Rating icon="❤️" value={score} onChange={setScore} />

// ✅ Custom icon bằng URL ảnh
<Rating icon="/static/images/heart.svg" value={score} onChange={setScore} />
```

---

### RULE-RATING-05: Sử dụng `max` để thay đổi số lượng icon; không render thủ công

Prop `max` quyết định số lượng icon hiển thị (mặc định là `5`). Không cần tự render danh sách icon bằng `Array.map`.

**Sai:**
```tsx
// ❌ Render thủ công — bỏ qua logic hover và select có sẵn
{Array(10).fill(0).map((_, i) => (
  <Rating key={i} value={i < score ? 1 : 0} max={1} />
))}
```

**Đúng:**
```tsx
// ✅ Sử dụng prop max để hiển thị 10 sao
<Rating max={10} value={score} onChange={setScore} />
```

---

### RULE-RATING-06: Sử dụng `iconSize` và `gap` để tuỳ chỉnh kích thước; không override bằng className

Prop `iconSize` (mặc định `30`) điều chỉnh kích thước mỗi icon (đơn vị px). Prop `gap` (mặc định `8`) điều chỉnh khoảng cách giữa các icon (đơn vị px). Không dùng className để thay đổi kích thước icon vì component sử dụng inline style.

**Sai:**
```tsx
// ❌ className không thể override inline style của icon size
<Rating className="[&>div]:w-[2.5rem] [&>div]:h-[2.5rem]" value={score} onChange={setScore} />
```

**Đúng:**
```tsx
// ✅ Sử dụng iconSize và gap
<Rating iconSize={40} gap={12} value={score} onChange={setScore} />
```

---

### RULE-RATING-07: Sử dụng `direction` để thay đổi hướng hiển thị

Prop `direction` nhận `"horizontal"` (mặc định) hoặc `"vertical"`. Không dùng className flex-col để thay đổi hướng vì component đã có logic xử lý direction.

**Sai:**
```tsx
// ❌ Override flex direction bằng className — conflict với internal logic
<Rating className="flex-col" value={score} onChange={setScore} />
```

**Đúng:**
```tsx
// ✅ Sử dụng prop direction
<Rating direction="vertical" value={score} onChange={setScore} />
```

---

### RULE-RATING-08: Sử dụng `disabled` để vô hiệu hoá tương tác

Prop `disabled` vô hiệu hoá click và hover trên tất cả các icon. Không tự thêm `pointer-events-none` hoặc wrap trong container có `opacity`.

**Sai:**
```tsx
// ❌ Tự thêm pointer-events-none — thiếu logic disabled internal
<div className="pointer-events-none opacity-50">
  <Rating value={score} onChange={setScore} />
</div>
```

**Đúng:**
```tsx
// ✅ Sử dụng prop disabled
<Rating disabled value={score} onChange={setScore} />
```
