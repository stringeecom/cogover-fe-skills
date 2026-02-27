## Cách sử dụng Avatar Component của ui-kit

Đường dẫn: `ui-kit/src/components/Avatar`

Props: `src`, `alt`, `size` (default 24, đơn vị px), `id` (string|null), `fallbackImage`, `badge` (AvatarProps trừ `badge`), `personnelDetailCardProps`, `personnelDetailPopperProps`. Hỗ trợ `React.ImgHTMLAttributes` và `ref`.

Fallback order: `src` → `fallbackImage` → ký tự đầu `alt` với gradient background (màu theo `id`)

```tsx
// Đầy đủ props (recommended)
<Avatar
  src={user.avatar}
  alt={user.name}          // bắt buộc: dùng làm initials khi không có ảnh
  size={32}                // px, dùng prop thay vì className w-/h-
  id={user.id}             // kích hoạt PersonnelDetailPopper + màu gradient duy nhất
  fallbackImage="/default-avatar.png"
/>

// Badge (góc dưới phải, mặc định 12px)
<Avatar
  src={user.avatar} alt={user.name} size={40} id={user.id}
  badge={{ src: statusIcon, alt: "Online", size: 16 }}
/>

// Tuỳ chỉnh popup
<Avatar
  src={user.avatar} alt={user.name} size={32} id={user.id}
  personnelDetailPopperProps={{ placement: "top" }}
/>
```

**DO:** Luôn truyền `alt` (dùng làm initials). Dùng `size` (số, px) thay vì className. Truyền `id` để có popup thông tin nhân sự và màu gradient riêng. Dùng `fallbackImage` thay vì tự xử lý `onError`. Dùng `badge` prop thay vì tự wrap.

**DON'T:** Tự tạo `<img>` hay `<div>` cho avatar. Dùng `className="w-8 h-8"` để set kích thước. Tự xử lý fallback với `onError`. Wrap Avatar trong Popper thủ công. Wrap badge trong div relative/absolute.

| size (px) | Dùng cho |
|-----------|----------|
| `24` | Danh sách, comment (mặc định) |
| `32` | Header, sidebar |
| `40` | Profile card |
| `48+` | Trang chi tiết profile |
