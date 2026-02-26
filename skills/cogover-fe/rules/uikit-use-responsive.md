---
title: Cách sử dụng hook useResponsive trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai isMobile vs isMobileDevice hoặc viết window.innerWidth trực tiếp gây hành vi responsive không nhất quán
tags: responsive, breakpoint, mobile, hook, ui-kit
---

## Cách sử dụng hook useResponsive trong ui-kit

Đường dẫn hook: `ui-kit/src/hooks/useResponsive`

## Các giá trị trả về

| Giá trị          | Điều kiện                          |
| ---------------- | ---------------------------------- |
| `isDesktop`      | `width >= 1024`                    |
| `isTablet`       | `768 < width < 1024`               |
| `isMobile`       | `width <= 767`                     |
| `isAuthDesktop`  | `width >= 1025`                    |
| `isAuthTablet`   | `768 < width < 1025`               |
| `isMobileDevice` | thiết bị có chạm VÀ `width < 1367` |
| `isLandscape`    | `width > height`                   |
| `height`         | chiều cao viewport hiện tại (px)   |
| `isMaxWidth(n)`  | `width <= n`                       |

### RULE-RESP-01: Sử dụng `isMobileDevice` cho phát hiện chạm, `isMobile` cho độ rộng viewport

- `isMobile`: độ rộng viewport `<= 767` — chỉ là breakpoint layout
- `isMobileDevice`: khả năng chạm thực tế (`ontouchstart` hoặc `maxTouchPoints > 0`) VÀ `width < 1367` — sử dụng cho quyết định tương tác (vô hiệu hóa hover, điều chỉnh xử lý sự kiện)

```tsx
// ✅ Phát hiện thiết bị chạm
const { isMobileDevice } = useResponsive();

// ✅ Phát hiện layout hẹp
const { isMobile } = useResponsive();
```

### RULE-RESP-02: Sử dụng `isMaxWidth` cho các breakpoints tùy chỉnh

**Sai:**

```tsx
// ❌ Truy cập window trực tiếp, không reactive
const isSmall = window.innerWidth <= 480;
```

**Đúng:**

```tsx
// ✅ Reactive, cập nhật khi resize
const { isMaxWidth } = useResponsive();
const isSmall = isMaxWidth(480);
```

### RULE-RESP-03: Không bao giờ viết so sánh `window.innerWidth` trực tiếp trong components

Luôn sử dụng `useResponsive`. Đọc `window.innerWidth` trực tiếp không reactive với các sự kiện resize.

### RULE-RESP-04: Sử dụng `isAuthDesktop`/`isAuthTablet` cho breakpoints layout auth

Các layouts auth sử dụng breakpoints hơi khác (khác 1px so với chuẩn):

| Chuẩn                   | Tương đương Auth            |
| ----------------------- | --------------------------- |
| `isDesktop` (`>= 1024`) | `isAuthDesktop` (`>= 1025`) |
| `isTablet` (`768–1023`) | `isAuthTablet` (`769–1024`) |
