## AvatarField Component (`uikit-avatar-field`)

Đường dẫn: `ui-kit/src/components/AvatarField`

- **AvatarField**: Hiển thị avatar từ `ImageFieldResponse` hoặc URL string. Kế thừa tất cả props của `Avatar` (`size`, `fallbackImage`, `badge`, `id`, `personnelDetailCardProps`, ...) và `ref` forwarding.
- **ImageField** (default export): Hiển thị hình ảnh `<img>` từ `ImageFieldResponse`, kế thừa `React.ImgHTMLAttributes`. Tự hiển thị placeholder khi không có URL.

### Props riêng

| Prop | Component | Kiểu | Mô tả |
|------|-----------|------|-------|
| `src` | AvatarField | `ImageResponse \| null \| undefined` | Chấp nhận cả URL string lẫn ImageFieldResponse |
| `data` | ImageField | `CamelCasedPropertiesDeep<ImageFieldResponse> \| ImageFieldResponse` | Dữ liệu từ API |
| `resolutionSize` | Cả hai | `ResolutionSizes` | Kích thước resolution |

### ResolutionSizes

| Giá trị | Kích thước | Dùng cho |
|---------|-----------|----------|
| `"tiny"` | 128x128 | Avatar nhỏ 24-32px |
| `"small"` | 256x256 | Avatar trung bình 40-64px |
| `"medium"` | 512x512 | Hình ảnh trung bình |
| `"large"` | 1024x1024 | Hình ảnh lớn trong bài viết |
| `"original"` | Gốc | Hiển thị kích thước gốc |

---

### RULE-AVATARFIELD-01: Dùng `AvatarField` — không tự xử lý URL từ `ImageFieldResponse`

```tsx
// ❌ Mất fallback, mất resolution
const avatarUrl = userData.avatar?.url?.replace("{file_id}", userData.avatar.file_id);
<Avatar src={avatarUrl} size={32} alt={userData.name} />

// ✅
<AvatarField src={userData.avatar} resolutionSize="small" size={32} alt={userData.name} />
```

---

### RULE-AVATARFIELD-02: Chọn `resolutionSize` phù hợp với kích thước hiển thị

```tsx
// ❌ Resolution "original" cho avatar 24px — lãng phí băng thông
<AvatarField src={user.avatar} resolutionSize="original" size={24} alt={user.name} />

// ✅
<AvatarField src={user.avatar} resolutionSize="tiny" size={24} alt={user.name} />    // 24-32px
<AvatarField src={user.avatar} resolutionSize="small" size={48} alt={user.name} />   // 40-64px
<AvatarField src={user.avatar} resolutionSize="medium" size={128} alt={user.name} /> // lớn hơn
```

---

### RULE-AVATARFIELD-03: Dùng `ImageField` cho hình ảnh thông thường (không phải avatar tròn)

```tsx
// ❌ AvatarField sẽ bị bo tròn
<AvatarField src={product.image} resolutionSize="medium" size={120} />

// ✅
import ImageField from "ui-kit/src/components/AvatarField";
<ImageField data={product.image} resolutionSize="medium" className="w-[7.5rem] h-[7.5rem] object-cover" />
```

---

### RULE-AVATARFIELD-04: Luôn truyền `alt` — dùng làm fallback text khi không có ảnh

```tsx
// ❌ Fallback hiển thị rỗng khi không có ảnh
<AvatarField src={user.avatar} resolutionSize="small" size={32} />

// ✅
<AvatarField src={user.avatar} resolutionSize="small" size={32} alt={user.name} />
```
