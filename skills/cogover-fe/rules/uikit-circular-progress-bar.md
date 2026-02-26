---
title: Cách sử dụng component CircularProgressBar trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến hiển thị phần trăm sai, kích thước vòng tròn không đúng hoặc chế độ unlimited không hoạt động
tags: progress, circular, percentage, unlimited, ui-kit
---

## Cách sử dụng component CircularProgressBar trong ui-kit

Đường dẫn component: `ui-kit/src/components/CircularProgressBar`

Props: `percentage` (bắt buộc), `color`, `isUnlimited`, `width`, `height`, `classFontSize`.

---

### RULE-CPB-01: `percentage` là prop duy nhất bắt buộc — luôn truyền giá trị hợp lệ từ 0 đến 100

Component sử dụng `percentage / 100` để tính toán `strokeDashoffset`, nên giá trị ngoài phạm vi `[0, 100]` sẽ gây ra hiển thị sai (vòng tròn tràn hoặc âm). Luôn đảm bảo giá trị hợp lệ trước khi truyền vào.

**Sai:**

```tsx
// ❌ Thiếu percentage — lỗi TypeScript
<CircularProgressBar color="#4caf50" />

// ❌ Giá trị có thể là NaN hoặc Infinity khi totalCount là 0
<CircularProgressBar percentage={completedCount / totalCount * 100} />

// ❌ Giá trị vượt quá 100 — vòng tròn hiển thị sai
<CircularProgressBar percentage={150} />
```

**Đúng:**

```tsx
// ✅ Luôn bảo vệ chống lại các giá trị không hợp lệ
const percentage = totalCount > 0 ? Math.min((completedCount / totalCount) * 100, 100) : 0;

<CircularProgressBar percentage={percentage} />
```

---

### RULE-CPB-02: Khi bật `isUnlimited`, vòng tròn luôn hiển thị đầy và bỏ qua giá trị `percentage`

Khi `isUnlimited={true}`, component luôn vẽ vòng tròn đầy (progress = 1) với gradient màu tím, và hiển thị icon infinity cùng text "Unlimited" thay vì số phần trăm. Giá trị `percentage` và `color` đều bị bỏ qua.

**Sai:**

```tsx
// ❌ percentage và color bị bỏ qua khi isUnlimited=true — gây nhầm lẫn cho người đọc code
<CircularProgressBar percentage={50} color="#ff0000" isUnlimited={true} />
```

**Đúng:**

```tsx
// ✅ Chỉ truyền isUnlimited — percentage vẫn bắt buộc bởi TypeScript nhưng giá trị bị bỏ qua
<CircularProgressBar percentage={0} isUnlimited={true} />

// ✅ Chế độ bình thường — hiển thị phần trăm với màu tùy chỉnh
<CircularProgressBar percentage={75} color="#4caf50" />
```

---

### RULE-CPB-03: Luôn truyền `width` và `height` cùng giá trị để vòng tròn không bị méo

Component sử dụng `Math.min(width, height) / 2 - strokeWidth / 2` để tính bán kính. Nếu `width` và `height` khác nhau, vòng tròn vẫn tròn nhưng sẽ bị lệch tâm trong SVG. Mặc định cả hai đều là `60`.

**Sai:**

```tsx
// ❌ width và height khác nhau — vòng tròn bị lệch tâm trong SVG
<CircularProgressBar percentage={50} width={80} height={40} />
```

**Đúng:**

```tsx
// ✅ Sử dụng giá trị mặc định (60x60)
<CircularProgressBar percentage={50} />

// ✅ Tùy chỉnh kích thước — giữ width và height bằng nhau
<CircularProgressBar percentage={50} width={48} height={48} />
```

---

### RULE-CPB-04: Sử dụng `classFontSize` để điều chỉnh kích thước chữ phần trăm khi thay đổi kích thước vòng tròn

Khi giảm kích thước vòng tròn (ví dụ `width={48} height={48}`), text phần trăm bên trong có thể bị tràn. Sử dụng `classFontSize` để truyền class CSS điều chỉnh kích thước chữ phù hợp.

**Sai:**

```tsx
// ❌ Vòng tròn nhỏ nhưng text vẫn giữ kích thước mặc định — có thể bị tràn
<CircularProgressBar percentage={100} width={40} height={40} />

// ❌ Sử dụng wrapper để override font-size — không ảnh hưởng đến thẻ strong bên trong
<div style={{ fontSize: 12 }}>
  <CircularProgressBar percentage={50} width={40} height={40} />
</div>
```

**Đúng:**

```tsx
// ✅ Truyền classFontSize để điều chỉnh kích thước chữ phù hợp với vòng tròn nhỏ
<CircularProgressBar
  percentage={100}
  width={48}
  height={48}
  classFontSize="!text-[12px]"
/>
```

---

### RULE-CPB-05: Sử dụng prop `color` để thay đổi màu thanh progress — không override qua CSS bên ngoài

`color` kiểm soát màu stroke của vòng tròn progress. Mặc định là `#4caf50` (xanh lá). Không sử dụng CSS bên ngoài để thay đổi màu vì component sử dụng inline `stroke` attribute trên SVG.

**Sai:**

```tsx
// ❌ CSS bên ngoài không override được stroke attribute trên SVG
<div style={{ color: "#ff0000" }}>
  <CircularProgressBar percentage={50} />
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng prop color trực tiếp
<CircularProgressBar percentage={50} color="#6366f1" />

// ✅ Sử dụng màu mặc định (#4caf50)
<CircularProgressBar percentage={50} />
```

---

### RULE-CPB-06: Gradient ID `gradientStroke` là cố định — tránh render nhiều instance `isUnlimited` trên cùng trang

Khi `isUnlimited={true}`, component tạo một `<linearGradient id="gradientStroke">` bên trong SVG. Nếu có nhiều instance cùng sử dụng `isUnlimited`, các gradient ID bị trùng lặp và trình duyệt có thể chỉ sử dụng gradient của instance đầu tiên.

**Sai:**

```tsx
// ❌ Nhiều instance unlimited trên cùng trang — gradient ID bị trùng
<CircularProgressBar percentage={0} isUnlimited={true} />
<CircularProgressBar percentage={0} isUnlimited={true} />
```

**Đúng:**

```tsx
// ✅ Chỉ sử dụng một instance unlimited trên cùng trang
<CircularProgressBar percentage={0} isUnlimited={true} />

// ✅ Các instance khác sử dụng chế độ bình thường
<CircularProgressBar percentage={75} color="#4caf50" />
```
