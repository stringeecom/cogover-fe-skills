---
title: Cách sử dụng component LazyImage trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến hình ảnh bị lỗi không có fallback, thiếu lazy loading và xử lý lỗi thủ công trùng lặp
tags: image, lazy, fallback, img, ui-kit
---

## Cách sử dụng component LazyImage trong ui-kit

Đường dẫn component: `ui-kit/src/components/LazyImage`

Props: `src`, `fallbackSrc`. Hỗ trợ tất cả `React.ImgHTMLAttributes<HTMLImageElement>` native và forwarding `ref`.

Wrap một `<img>` native với `loading="lazy"` và hành vi fallback-on-error tự động. Khi `src` không tải được, nó tự động chuyển sang `fallbackSrc`.

### RULE-LAZY-IMG-01: Sử dụng `LazyImage` thay vì `<img>` thô cho tất cả images

KHÔNG sử dụng tag `<img>` thô. Luôn sử dụng `LazyImage` để có lazy loading và hỗ trợ fallback ngay lập tức.

**Sai:**

```tsx
// ❌ Img thô — không có lazy loading, không có fallback
<img src={user.avatar} alt="avatar" />
```

**Đúng:**

```tsx
// ✅
<LazyImage src={user.avatar} alt="avatar" />
```

### RULE-LAZY-IMG-02: Sử dụng `fallbackSrc` cho degradation graceful khi images bị lỗi

Khi một URL hình ảnh có thể không hợp lệ hoặc tạm thời không khả dụng, luôn cung cấp `fallbackSrc`. KHÔNG tự xử lý sự kiện `onError` chỉ để swap src.

**Sai:**

```tsx
// ❌ onError thủ công trùng lặp logic fallback có sẵn
<LazyImage
  src={user.avatar}
  onError={(e) => {
    (e.target as HTMLImageElement).src = defaultAvatar;
  }}
/>
```

**Đúng:**

```tsx
// ✅ fallbackSrc tự xử lý swap
<LazyImage src={user.avatar} fallbackSrc={defaultAvatar} alt="avatar" />
```

### RULE-LAZY-IMG-03: Không override attribute `loading`

`LazyImage` set `loading="lazy"` mặc định. KHÔNG truyền `loading="eager"` trừ khi có yêu cầu above-the-fold cụ thể (ví dụ: LCP image).

**Sai:**

```tsx
// ❌ Override lazy loading vô lý do
<LazyImage src={banner} loading="eager" />
```

**Đúng:**

```tsx
// ✅ Chỉ sử dụng eager cho các hình ảnh above-the-fold quan trọng
<LazyImage src={heroBanner} loading="eager" alt="Hero" />

// ✅ Mặc định là lazy — không cần chỉ định
<LazyImage src={thumbnail} alt="Thumbnail" />
```

### RULE-LAZY-IMG-04: Chỉ truyền `onError` cho các side effects bổ sung, không dùng để swap src

`onError` được forward và gọi sau khi logic fallback nội bộ chạy. Chỉ sử dụng nó cho logging hoặc analytics, không dùng để thay đổi `src`.

```tsx
// ✅ onError chỉ cho side effect
<LazyImage
  src={imageUrl}
  fallbackSrc={placeholder}
  onError={() => trackBrokenImage(imageUrl)}
/>
```

### RULE-LAZY-IMG-05: Không sử dụng `fallbackSrc` như src chính

`fallbackSrc` chỉ hiển thị khi `src` thất bại hoặc rỗng. Nếu không có hình ảnh chính, truyền placeholder trực tiếp vào `src` thay vào đó.

**Sai:**

```tsx
// ❌ src rỗng — fallbackSrc không bao giờ kích hoạt trừ khi src lỗi
<LazyImage src="" fallbackSrc={placeholder} />
```

**Đúng:**

```tsx
// ✅ Không có hình ảnh chính — truyền placeholder như src trực tiếp
<LazyImage src={imageUrl || placeholder} alt="image" />

// ✅ Hoặc sử dụng fallbackSrc khi URL có thể thất bại
<LazyImage src={imageUrl} fallbackSrc={placeholder} alt="image" />
```
