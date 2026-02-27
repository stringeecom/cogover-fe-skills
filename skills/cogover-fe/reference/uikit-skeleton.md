---
title: Cách sử dụng Skeleton Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng Skeleton sai dẫn đến việc chọn variant không phù hợp, tự tạo animation loading thủ công và không tận dụng được className để tuỳ chỉnh kích thước
tags: skeleton, loading, placeholder, wave, animation, ui-kit
---

## Cách sử dụng Skeleton Component của ui-kit

Đường dẫn component: `ui-kit/src/components/Skeleton`

Props: `variant`, `className`. Hỗ trợ tất cả `React.HTMLAttributes<HTMLElement>`.

---

### RULE-SKEL-01: Chọn `variant` phù hợp với hình dạng nội dung cần hiển thị

Skeleton hỗ trợ 4 variant: `"text"` (mặc định), `"circular"`, `"rectangular"` và `"rounded"`. Chọn variant tương ứng với hình dạng của nội dung thực tế sẽ thay thế skeleton.

**Sai:**

```tsx
// ❌ Dùng variant text cho avatar hình tròn
<Skeleton variant="text" className={cx("w-[2.5rem] h-[2.5rem] rounded-full")} />

// ❌ Dùng variant rectangular cho đoạn văn bản
<Skeleton variant="rectangular" />
```

**Đúng:**

```tsx
// ✅ Dùng variant circular cho avatar
<Skeleton variant="circular" className={cx("w-[2.5rem] h-[2.5rem]")} />

// ✅ Dùng variant text (mặc định) cho dòng văn bản
<Skeleton className={cx("w-2/3 h-[0.5rem]")} />

// ✅ Dùng variant rounded cho card hoặc khối nội dung có bo góc
<Skeleton variant="rounded" className={cx("w-5/6 h-[2.5rem]")} />

// ✅ Dùng variant rectangular cho hình ảnh hoặc khối nội dung góc vuông
<Skeleton variant="rectangular" className={cx("w-full h-[5rem]")} />
```

---

### RULE-SKEL-02: Không truyền rõ ràng giá trị mặc định cho `variant`

`variant` mặc định là `"text"`. Chỉ truyền khi cần thay đổi giá trị mặc định.

**Sai:**

```tsx
// ❌ Thừa — variant đã có giá trị mặc định là "text"
<Skeleton variant="text" className={cx("w-full h-[0.75rem]")} />
```

**Đúng:**

```tsx
// ✅ Bỏ qua khi dùng giá trị mặc định
<Skeleton className={cx("w-full h-[0.75rem]")} />

// ✅ Chỉ truyền khi cần thay đổi
<Skeleton variant="circular" className={cx("w-[2.5rem] h-[2.5rem]")} />
```

---

### RULE-SKEL-03: Tuỳ chỉnh kích thước bằng `className`, không dùng inline style

Skeleton cho phép tuỳ chỉnh kích thước thông qua `className`. Không sử dụng inline `style` để đặt kích thước.

**Sai:**

```tsx
// ❌ Dùng inline style để đặt kích thước
<Skeleton style={{ width: "200px", height: "12px" }} />
```

**Đúng:**

```tsx
// ✅ Dùng className với Tailwind để tuỳ chỉnh kích thước
<Skeleton className={cx("w-full h-[0.75rem]")} />

// ✅ Dùng kích thước tương đối
<Skeleton className={cx("w-2/3 h-[0.5rem]")} />
```

---

### RULE-SKEL-04: Không tự tạo animation loading thủ công, sử dụng Skeleton có sẵn

Skeleton đã tích hợp sẵn hiệu ứng wave animation. Không tự tạo các hiệu ứng loading tương tự bằng CSS hoặc thư viện khác.

**Sai:**

```tsx
// ❌ Tự tạo skeleton loading bằng CSS thủ công
<div className={cx("animate-pulse bg-gray-200 rounded", "w-full h-[0.75rem]")} />

// ❌ Dùng thư viện khác để tạo skeleton
import ContentLoader from "react-content-loader";
<ContentLoader>
  <rect width="200" height="12" />
</ContentLoader>
```

**Đúng:**

```tsx
// ✅ Sử dụng Skeleton component có sẵn với wave animation tích hợp
<Skeleton className={cx("w-full h-[0.75rem]")} />
```

---

### RULE-SKEL-05: Tổ hợp nhiều Skeleton để tạo layout loading phức tạp

Khi cần hiển thị trạng thái loading cho một khối nội dung phức tạp (ví dụ: danh sách, card), tổ hợp nhiều Skeleton với các variant khác nhau thay vì dùng một Skeleton duy nhất.

**Sai:**

```tsx
// ❌ Dùng một Skeleton lớn duy nhất cho toàn bộ khối nội dung
<Skeleton variant="rectangular" className={cx("w-full h-[12.5rem]")} />
```

**Đúng:**

```tsx
// ✅ Tổ hợp nhiều Skeleton để mô phỏng layout thực tế
<div className={cx("flex gap-[0.75rem] items-center")}>
  <Skeleton variant="circular" className={cx("w-[2.5rem] h-[2.5rem]")} />
  <div className={cx("flex flex-col gap-[0.5rem] w-5/6")}>
    <Skeleton className={cx("w-2/3 h-[0.5rem]")} />
    <Skeleton variant="rounded" className={cx("w-5/6 h-[2.5rem]")} />
  </div>
</div>
```

---

## Tham chiếu style

| Variant | Bo góc | Kích thước mặc định |
|---------|--------|---------------------|
| `text` | `rounded` | `w-[200px]` max-width 100% |
| `circular` | `rounded-full` | `w-[40px] h-[40px]` |
| `rectangular` | Không | `w-[200px] h-[80px]` max 100% |
| `rounded` | `rounded` | `w-[200px] h-[80px]` max 100% |

| Thành phần | Style |
|------------|-------|
| Background | `bg-skeleton` |
| Wave animation | `bg-skeleton-wave` với `animate-surf` |
