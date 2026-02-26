---
title: Cách sử dụng Spinner Component của ui-kit
impact: LOW
impactDescription: Sử dụng Spinner sai dẫn đến việc tuỳ chỉnh kích thước không đúng cách, trùng lặp animation, hoặc không tận dụng được màu brand tự động
tags: spinner, loading, animation, ui-kit
---

## Cách sử dụng Spinner Component của ui-kit

Đường dẫn component: `ui-kit/src/components/Spinner`

Export: `SpinnerComponent` (từ `ui-kit/src/components/index.ts`)

Props: Hỗ trợ tất cả `React.SVGProps<SVGSVGElement>`, bao gồm `className`, `width`, `height`, v.v.

Đặc điểm: Spinner là một SVG hình tròn với animation xoay (`animate-spin`) được tích hợp sẵn. Màu stroke tự động sử dụng **brand color** từ `commonSettings` hoặc `localStorage`.

---

### RULE-SPINNER-01: Sử dụng `SpinnerComponent` thay vì tự tạo spinner với FontAwesome hoặc CSS

Khi cần hiển thị trạng thái loading dạng spinner, sử dụng `SpinnerComponent` từ ui-kit. Không tự tạo spinner bằng FontAwesome icon với `animate-spin` vì sẽ không đồng bộ với brand color.

**Sai:**

```tsx
// ❌ Tự tạo spinner — không sử dụng brand color, không nhất quán UI
<FontAwesomeIcon icon={faSpinner} className="animate-spin" />

// ❌ Tự tạo spinner bằng div — dư thừa code
<div className="animate-spin rounded-full border-2 border-primary-main border-t-transparent w-4 h-4" />
```

**Đúng:**

```tsx
// ✅ Sử dụng SpinnerComponent — tự động đồng bộ brand color
import { SpinnerComponent } from "ui-kit";

<SpinnerComponent />
```

---

### RULE-SPINNER-02: Không thêm `animate-spin` vào `className` — animation đã được tích hợp sẵn

Spinner đã có sẵn class `animate-spin` bên trong. Việc thêm lại sẽ dư thừa.

**Sai:**

```tsx
// ❌ Dư thừa — animate-spin đã được tích hợp sẵn trong component
<SpinnerComponent className={cx("animate-spin")} />
```

**Đúng:**

```tsx
// ✅ Không cần thêm animate-spin
<SpinnerComponent />
```

---

### RULE-SPINNER-03: Tuỳ chỉnh kích thước bằng `className` với `w-*` và `h-*`, không dùng prop `width`/`height`

Kích thước mặc định của Spinner là `16x16` (pixel, thuộc tính SVG). Khi cần thay đổi kích thước, sử dụng Tailwind class `w-*` và `h-*` thông qua `className` để đảm bảo responsive và nhất quán với hệ thống design.

**Sai:**

```tsx
// ❌ Dùng prop SVG — không nhất quán với hệ thống Tailwind
<SpinnerComponent width={60} height={60} />

// ❌ Dùng inline style — không nhất quán
<SpinnerComponent style={{ width: 60, height: 60 }} />
```

**Đúng:**

```tsx
// ✅ Dùng Tailwind class để tuỳ chỉnh kích thước
<SpinnerComponent className={cx("w-[3.75rem] h-[3.75rem]")} />

// ✅ Kích thước mặc định (16x16) — không cần thêm gì
<SpinnerComponent />
```

---

### RULE-SPINNER-04: Khi dùng trong Button, sử dụng prop `loading` của Button thay vì chèn Spinner thủ công

Button component đã có prop `loading` tích hợp sẵn để xử lý trạng thái loading. Không chèn `SpinnerComponent` vào children hoặc icon của Button.

**Sai:**

```tsx
// ❌ Chèn Spinner thủ công vào Button — dư thừa
<Button disabled={isLoading}>
  {isLoading ? <SpinnerComponent /> : "Lưu"}
</Button>

// ❌ Chèn Spinner vào startIcon — không đúng cách
<Button startIcon={isLoading ? <SpinnerComponent /> : <SaveIcon />}>
  Lưu
</Button>
```

**Đúng:**

```tsx
// ✅ Sử dụng prop loading của Button
<Button loading={isLoading}>Lưu</Button>
```

---

## Tham chiếu

| Thuộc tính | Giá trị |
|-----------|---------|
| Kích thước mặc định | `16x16` (SVG viewBox `0 0 16 16`) |
| Animation | `animate-spin` (tích hợp sẵn) |
| Màu stroke | Brand color tự động (từ `commonSettings` hoặc `localStorage`) |
| Export name | `SpinnerComponent` |
