---
title: Cách sử dụng Avatar Component của ui-kit
impact: HIGH
impactDescription: Sử dụng Avatar sai dẫn đến việc fallback không hoạt động, badge hiển thị sai, và popup thông tin nhân sự không xuất hiện
tags: avatar, image, badge, personnel, ui-kit
---

## Cách sử dụng Avatar Component của ui-kit

Đường dẫn component: `ui-kit/src/components/Avatar`

Props: `src`, `alt`, `size`, `id`, `fallbackImage`, `badge`, `personnelDetailCardProps`, `personnelDetailPopperProps`. Hỗ trợ tất cả `React.ImgHTMLAttributes` (ngoại trừ `id` được override thành `string | null`) và `ref` forwarding.

---

### RULE-AVT-01: Luôn truyền `alt` để hiển thị tên viết tắt khi không có ảnh

Khi không có `src` hoặc ảnh load thất bại và không có `fallbackImage`, Avatar sẽ hiển thị ký tự đầu tiên của `alt` làm tên viết tắt (initials). Nếu không truyền `alt`, Avatar sẽ hiển thị trống.

**Sai:**

```tsx
// ❌ Không có alt — Avatar sẽ hiển thị trống khi không có ảnh
<Avatar src={user.avatar} size={32} />
```

**Đúng:**

```tsx
// ✅ alt cung cấp tên viết tắt khi không có ảnh
<Avatar src={user.avatar} alt={user.name} size={32} />
```

---

### RULE-AVT-02: Sử dụng `fallbackImage` thay vì tự xử lý onError

Avatar đã có sẵn cơ chế xử lý lỗi: khi ảnh load thất bại, nó tự động chuyển sang `fallbackImage`. Không cần tự viết logic onError để thay thế ảnh.

**Sai:**

```tsx
// ❌ Tự xử lý fallback — duplicate logic đã có sẵn
const [imgSrc, setImgSrc] = useState(user.avatar);

<img
  src={imgSrc}
  onError={() => setImgSrc("/default-avatar.png")}
  className="rounded-full w-8 h-8"
/>
```

**Đúng:**

```tsx
// ✅ fallbackImage tự động xử lý khi src load thất bại
<Avatar
  src={user.avatar}
  alt={user.name}
  fallbackImage="/default-avatar.png"
  size={32}
/>
```

---

### RULE-AVT-03: Sử dụng `size` (số, đơn vị px) thay vì className để đặt kích thước

Avatar sử dụng CSS variable `--size` từ prop `size` (mặc định 24) để đặt kích thước đồng nhất cho cả ảnh và initials. Không dùng className để override kích thước.

**Sai:**

```tsx
// ❌ Override kích thước bằng className — gây xung đột với CSS variable nội bộ
<Avatar src={user.avatar} alt={user.name} className="w-10 h-10" />
```

**Đúng:**

```tsx
// ✅ size tính bằng pixel, Avatar tự tính toán kích thước
<Avatar src={user.avatar} alt={user.name} size={40} />
```

---

### RULE-AVT-04: Sử dụng prop `badge` để hiển thị badge, không tự wrap thêm Avatar

Prop `badge` nhận một object có cùng props như Avatar (ngoại trừ `badge`). Badge sẽ tự động hiển thị ở góc dưới bên phải với kích thước mặc định 12px.

**Sai:**

```tsx
// ❌ Tự wrap badge — sai vị trí và thiếu style nhất quán
<div className="relative">
  <Avatar src={user.avatar} alt={user.name} size={40} />
  <img src={statusIcon} className="absolute bottom-0 right-0 w-3 h-3 rounded-full" />
</div>
```

**Đúng:**

```tsx
// ✅ Badge được xử lý tự động với vị trí và style đúng
<Avatar
  src={user.avatar}
  alt={user.name}
  size={40}
  badge={{
    src: statusIcon,
    alt: "Online",
  }}
/>
```

Tùy chỉnh kích thước badge:

```tsx
// ✅ Badge với kích thước tùy chỉnh
<Avatar
  src={user.avatar}
  alt={user.name}
  size={40}
  badge={{
    src: statusIcon,
    alt: "Online",
    size: 16,
  }}
/>
```

---

### RULE-AVT-05: Truyền `id` để kích hoạt popup thông tin nhân sự và màu nền tự động

Prop `id` phục vụ hai mục đích:
1. Kích hoạt `PersonnelDetailPopper` — khi hover vào Avatar sẽ hiện popup thông tin nhân sự
2. Tạo màu gradient background dựa trên `id` khi hiển thị initials (mỗi `id` sẽ có màu khác nhau)

**Sai:**

```tsx
// ❌ Không truyền id — không có popup thông tin nhân sự, tất cả initials cùng màu
<Avatar src={user.avatar} alt={user.name} size={32} />
```

**Đúng:**

```tsx
// ✅ Truyền id để có popup và màu nền duy nhất
<Avatar
  src={user.avatar}
  alt={user.name}
  size={32}
  id={user.id}
/>
```

---

### RULE-AVT-06: Sử dụng `personnelDetailPopperProps` để tùy chỉnh popup, không tự wrap Popper

Khi cần tùy chỉnh hành vi popup thông tin nhân sự (vị trí, delay...), sử dụng `personnelDetailPopperProps`. Không tự wrap Avatar trong Popper.

**Sai:**

```tsx
// ❌ Tự wrap Popper — duplicate logic và mất tích hợp với PersonnelDetailCard
<Popper
  placement="top"
  render={() => <PersonnelDetailCard personnelId={user.id} />}
>
  {(params) => (
    <div {...params}>
      <Avatar src={user.avatar} alt={user.name} size={32} />
    </div>
  )}
</Popper>
```

**Đúng:**

```tsx
// ✅ Tùy chỉnh popup qua prop
<Avatar
  src={user.avatar}
  alt={user.name}
  size={32}
  id={user.id}
  personnelDetailPopperProps={{
    placement: "top",
  }}
/>
```

---

### RULE-AVT-07: Không tự tạo component avatar với img tag hoặc div

Avatar đã xử lý: lazy loading ảnh (`LazyImage`), fallback initials với gradient background, badge, popup thông tin nhân sự, và forward ref. Không tự tạo lại các tính năng này.

**Sai:**

```tsx
// ❌ Tự tạo avatar — thiếu lazy loading, fallback, popup thông tin nhân sự
<img
  src={user.avatar}
  alt={user.name}
  className="w-8 h-8 rounded-full object-cover"
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng Avatar component để có đầy đủ tính năng
<Avatar
  src={user.avatar}
  alt={user.name}
  size={32}
  id={user.id}
  fallbackImage="/default-avatar.png"
/>
```

---

## Tham chiếu kích thước

| size (px) | Mô tả |
|-----------|-------|
| `24` | Mặc định, dùng cho danh sách, comment |
| `32` | Dùng cho header, sidebar |
| `40` | Dùng cho profile card |
| `48+` | Dùng cho trang chi tiết profile |

## Cơ chế fallback

Thứ tự hiển thị của Avatar:
1. Hiển thị ảnh từ `src`
2. Nếu `src` load thất bại → hiển thị `fallbackImage`
3. Nếu không có `fallbackImage` hoặc `fallbackImage` cũng load thất bại → hiển thị ký tự đầu tiên của `alt` với gradient background (màu dựa trên `id`)
