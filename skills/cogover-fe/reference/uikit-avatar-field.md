---
title: Cách sử dụng AvatarField Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng AvatarField sai dẫn đến việc xử lý image response thủ công, chọn sai resolution và mất fallback avatar
tags: avatar, image, avatar-field, resolution, ui-kit
---

## Cách sử dụng AvatarField Component của ui-kit

Đường dẫn component: `ui-kit/src/components/AvatarField`

Component này bao gồm 2 thành phần chính:
- **AvatarField**: Hiển thị avatar từ `ImageFieldResponse` hoặc URL string, kế thừa tất cả props của `Avatar` (bao gồm `size`, `fallbackImage`, `badge`, `id`, `personnelDetailCardProps`, ...) và hỗ trợ `ref` forwarding.
- **ImageField** (default export từ `index.tsx`): Hiển thị hình ảnh từ `ImageFieldResponse`, kế thừa tất cả `React.ImgHTMLAttributes`. Khi không có URL hợp lệ, tự động hiển thị ảnh placeholder.

### Props riêng của AvatarField

| Prop | Kiểu | Mô tả |
|------|------|-------|
| `src` | `ImageResponse \| null \| undefined` | Dữ liệu ảnh - có thể là `ImageFieldResponse`, `CamelCasedPropertiesDeep<ImageFieldResponse>`, hoặc một URL string |
| `resolutionSize` | `ResolutionSizes` | Kích thước resolution cần hiển thị |

### Props riêng của ImageField

| Prop | Kiểu | Mô tả |
|------|------|-------|
| `data` | `CamelCasedPropertiesDeep<ImageFieldResponse> \| ImageFieldResponse` | Dữ liệu ảnh từ API response |
| `resolutionSize` | `ResolutionSizes` | Kích thước resolution cần hiển thị |

### Các giá trị `ResolutionSizes`

| Giá trị | Kích thước | Mục đích sử dụng |
|---------|-----------|------------------|
| `"tiny"` | 128x128 | Toolbar và các tính năng nhỏ |
| `"small"` | 256x256 | Avatar |
| `"medium"` | 512x512 | Hình ảnh trung bình |
| `"large"` | 1024x1024 | Hình ảnh lớn trong bài viết |
| `"original"` | Gốc | Hiển thị kích thước gốc |

---

### RULE-AVATARFIELD-01: Sử dụng `AvatarField` thay vì tự xử lý URL từ `ImageFieldResponse`

Khi cần hiển thị avatar từ dữ liệu API trả về (dạng `ImageFieldResponse`), sử dụng `AvatarField` để tự động xử lý việc trích xuất URL và chọn resolution. KHÔNG tự tay gọi URL rồi truyền vào `Avatar`.

**Sai:**
```tsx
// ❌ Tự xử lý URL thủ công - mất fallback, mất xử lý resolution
import { Avatar } from "ui-kit";

const avatarUrl = userData.avatar?.url?.replace("{file_id}", userData.avatar.file_id);
<Avatar src={avatarUrl} size={32} alt={userData.name} />
```

**Đúng:**
```tsx
// ✅ AvatarField tự động xử lý URL, resolution và fallback
import { AvatarField } from "ui-kit";

<AvatarField src={userData.avatar} resolutionSize="small" size={32} alt={userData.name} />
```

---

### RULE-AVATARFIELD-02: Chọn `resolutionSize` phù hợp với kích thước hiển thị

Chọn resolution phù hợp để tối ưu hiệu suất tải trang. Không dùng `"original"` hoặc `"large"` cho avatar nhỏ.

**Sai:**
```tsx
// ❌ Dùng resolution "original" cho avatar nhỏ 24px - lãng phí băng thông
<AvatarField src={user.avatar} resolutionSize="original" size={24} alt={user.name} />

// ❌ Dùng resolution "large" cho avatar 32px
<AvatarField src={user.avatar} resolutionSize="large" size={32} alt={user.name} />
```

**Đúng:**
```tsx
// ✅ Avatar nhỏ (24-32px) - dùng "tiny"
<AvatarField src={user.avatar} resolutionSize="tiny" size={24} alt={user.name} />

// ✅ Avatar trung bình (40-64px) - dùng "small"
<AvatarField src={user.avatar} resolutionSize="small" size={48} alt={user.name} />

// ✅ Hình ảnh lớn - dùng "medium" hoặc "large"
<AvatarField src={user.avatar} resolutionSize="medium" size={128} alt={user.name} />
```

---

### RULE-AVATARFIELD-03: `AvatarField` chấp nhận cả URL string lẫn `ImageFieldResponse`

Khi `src` là một URL string, `AvatarField` sẽ truyền thẳng vào `Avatar` mà không cần xử lý gì thêm. Không cần kiểm tra kiểu dữ liệu trước khi truyền.

**Sai:**
```tsx
// ❌ Kiểm tra kiểu thủ công không cần thiết
{typeof user.avatar === "string" ? (
    <Avatar src={user.avatar} size={32} alt={user.name} />
) : (
    <AvatarField src={user.avatar} resolutionSize="small" size={32} alt={user.name} />
)}
```

**Đúng:**
```tsx
// ✅ AvatarField tự động xử lý cả string URL lẫn ImageFieldResponse
<AvatarField src={user.avatar} resolutionSize="small" size={32} alt={user.name} />
```

---

### RULE-AVATARFIELD-04: Sử dụng `ImageField` khi cần hiển thị hình ảnh (không phải avatar)

`ImageField` (default export) hiển thị hình ảnh dạng `<img>` với placeholder khi không có URL. Sử dụng cho trường hợp hiển thị hình ảnh thông thường, không phải avatar tròn.

**Sai:**
```tsx
// ❌ Dùng AvatarField để hiển thị thumbnail hình ảnh - sẽ bị bo tròn
<AvatarField src={product.image} resolutionSize="medium" size={120} />
```

**Đúng:**
```tsx
// ✅ Dùng ImageField cho hình ảnh thông thường
import ImageField from "ui-kit/src/components/AvatarField";

<ImageField data={product.image} resolutionSize="medium" className="w-[7.5rem] h-[7.5rem] object-cover" />
```

---

### RULE-AVATARFIELD-05: `AvatarField` kế thừa tất cả props của `Avatar`

`AvatarField` hỗ trợ tất cả props của `Avatar` như `size`, `fallbackImage`, `badge`, `id`, `personnelDetailCardProps`, `personnelDetailPopperProps`, `className`, `alt`, ... Không cần wrap thêm `Avatar` bên ngoài.

**Sai:**
```tsx
// ❌ Wrap AvatarField trong Avatar - dư thừa và có thể gây lỗi
<Avatar size={32} badge={badgeProps}>
    <AvatarField src={user.avatar} resolutionSize="small" />
</Avatar>
```

**Đúng:**
```tsx
// ✅ Truyền trực tiếp các props của Avatar vào AvatarField
<AvatarField
    src={user.avatar}
    resolutionSize="small"
    size={32}
    alt={user.name}
    id={user.id}
    badge={badgeProps}
    fallbackImage="/images/default-avatar.png"
    personnelDetailCardProps={{ showEmail: true }}
/>
```

---

### RULE-AVATARFIELD-06: Truyền `alt` cho avatar để đảm bảo accessibility

`AvatarField` kế thừa `alt` từ `React.ImgHTMLAttributes`. Khi không có `src` hợp lệ, `Avatar` sẽ hiển thị viết tắt từ `alt` làm fallback. Luôn truyền `alt` để có fallback tốt.

**Sai:**
```tsx
// ❌ Thiếu alt - fallback sẽ hiển thị rỗng khi không có ảnh
<AvatarField src={user.avatar} resolutionSize="small" size={32} />
```

**Đúng:**
```tsx
// ✅ Có alt - fallback sẽ hiển thị chữ viết tắt từ tên
<AvatarField src={user.avatar} resolutionSize="small" size={32} alt={user.name} />
```
