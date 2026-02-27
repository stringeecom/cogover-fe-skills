## Skeleton Component (`uikit-skeleton`)

Đường dẫn: `ui-kit/src/components/Skeleton`

Props: `variant`, `className`. Hỗ trợ `React.HTMLAttributes<HTMLElement>`.

| Variant | Bo góc | Kích thước mặc định | Dùng cho |
|---------|--------|---------------------|----------|
| `text` (mặc định) | `rounded` | `w-[200px]` max 100% | Dòng văn bản |
| `circular` | `rounded-full` | `w-[40px] h-[40px]` | Avatar tròn |
| `rectangular` | Không | `w-[200px] h-[80px]` max 100% | Hình ảnh, khối góc vuông |
| `rounded` | `rounded` | `w-[200px] h-[80px]` max 100% | Card, khối có bo góc |

---

### RULE-SKEL-01: Chọn `variant` phù hợp với hình dạng nội dung

```tsx
// ❌ Sai variant
<Skeleton variant="text" className={cx("w-[2.5rem] h-[2.5rem] rounded-full")} />   // avatar
<Skeleton variant="rectangular" />                                                   // đoạn văn bản

// ✅
<Skeleton variant="circular" className={cx("w-[2.5rem] h-[2.5rem]")} />   // avatar
<Skeleton className={cx("w-2/3 h-[0.5rem]")} />                            // văn bản (text là mặc định)
<Skeleton variant="rounded" className={cx("w-5/6 h-[2.5rem]")} />         // card bo góc
<Skeleton variant="rectangular" className={cx("w-full h-[5rem]")} />      // hình ảnh
```

---

### RULE-SKEL-02: Dùng `className` để tùy chỉnh kích thước — không dùng inline style

```tsx
// ❌
<Skeleton style={{ width: "200px", height: "12px" }} />

// ✅
<Skeleton className={cx("w-full h-[0.75rem]")} />
<Skeleton className={cx("w-2/3 h-[0.5rem]")} />
```

---

### RULE-SKEL-03: Không tự tạo animation loading — dùng Skeleton có wave animation tích hợp

```tsx
// ❌
<div className={cx("animate-pulse bg-gray-200 rounded w-full h-[0.75rem]")} />

// ✅
<Skeleton className={cx("w-full h-[0.75rem]")} />
```

---

### RULE-SKEL-04: Tổ hợp nhiều Skeleton để tạo layout loading phức tạp

```tsx
// ❌ Một Skeleton lớn cho toàn bộ khối
<Skeleton variant="rectangular" className={cx("w-full h-[12.5rem]")} />

// ✅ Mô phỏng layout thực tế
<div className={cx("flex gap-[0.75rem] items-center")}>
  <Skeleton variant="circular" className={cx("w-[2.5rem] h-[2.5rem]")} />
  <div className={cx("flex flex-col gap-[0.5rem] w-5/6")}>
    <Skeleton className={cx("w-2/3 h-[0.5rem]")} />
    <Skeleton variant="rounded" className={cx("w-5/6 h-[2.5rem]")} />
  </div>
</div>
```
