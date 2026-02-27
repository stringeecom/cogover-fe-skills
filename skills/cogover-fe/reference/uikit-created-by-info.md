---
title: Cách sử dụng component CreatedByInfo trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến render avatar không nhất quán, thiếu tooltip khi overflow và indicator system sai
tags: avatar, creator, user, system, automation, ui-kit
---

## Cách sử dụng component CreatedByInfo trong ui-kit

Đường dẫn component: `ui-kit/src/components/CreatedByInfo`

Props: `name`, `avatar`, `id`, `isSystem`, `size`, `resolutionSize`.

### RULE-CREATED-BY-01: Sử dụng `isSystem` cho Automation Cogover, không dùng markup thủ công

Khi người tạo là hệ thống/automation entity, luôn sử dụng prop `isSystem`. KHÔNG tự render logo Cogover hoặc hardcode text "Automation Cogover".

**Sai:**

```tsx
// ❌ Render system thủ công trùng lặp logic có sẵn
<div className={cx("flex items-center gap-[0.5rem]")}>
  <img src={cogoverLogo} className={cx("w-[1.5rem] h-[1.5rem] rounded-full")} />
  <span>Automation Cogover</span>
</div>
```

**Đúng:**

```tsx
// ✅
<CreatedByInfo isSystem />
```

### RULE-CREATED-BY-02: Luôn truyền `id` để đảm bảo màu avatar fallback nhất quán

Khi `avatar` thiếu hoặc không tải được, màu avatar fallback được tạo từ `id`. Bỏ qua `id` dẫn đến màu không nhất quán giữa các lần render.

**Sai:**

```tsx
// ❌ Thiếu id — màu fallback có thể không nhất quán
<CreatedByInfo name={user.fullName} avatar={user.avatarUrl} />
```

**Đúng:**

```tsx
// ✅
<CreatedByInfo id={user.id} name={user.fullName} avatar={user.avatarUrl} />
```

### RULE-CREATED-BY-03: Truyền trực tiếp object `ImageResponse` khi avatar đến từ API

Khi API trả về object `ImageResponse`, truyền trực tiếp nó vào `avatar`. KHÔNG tự trích xuất URL string — sử dụng `resolutionSize` để kiểm soát chất lượng hình ảnh thay vào đó.

**Sai:**

```tsx
// ❌ Tự trích xuất URL bỏ qua xử lý resolution
<CreatedByInfo id={user.id} name={user.fullName} avatar={user.avatar.url} />
```

**Đúng:**

```tsx
// ✅ Truyền toàn bộ object ImageResponse
<CreatedByInfo
  id={user.id}
  name={user.fullName}
  avatar={user.avatar}
  resolutionSize="small"
/>
```

### RULE-CREATED-BY-04: Không wrap với custom tooltip — SmartTooltip đã được tích hợp sẵn

`CreatedByInfo` đã áp dụng `SmartTooltip` trên name mà chỉ kích hoạt khi overflow. KHÔNG thêm `Tooltip` hoặc `SmartTooltip` bên ngoài.

**Sai:**

```tsx
// ❌ Tooltip kép — tooltip ngoài xung đột với SmartTooltip có sẵn
<Tooltip content={user.fullName}>
  <CreatedByInfo id={user.id} name={user.fullName} avatar={user.avatarUrl} />
</Tooltip>
```

**Đúng:**

```tsx
// ✅ Tooltip được xử lý nội bộ
<CreatedByInfo id={user.id} name={user.fullName} avatar={user.avatarUrl} />
```

### RULE-CREATED-BY-05: Không tự tạo lại layout này bằng Avatar + name span thủ công

Khi hiển thị người tạo với avatar và name, luôn sử dụng `CreatedByInfo`. KHÔNG tự kết hợp `Avatar` + `<span>` thủ công.

**Sai:**

```tsx
// ❌ Kết hợp thủ công mất SmartTooltip, sizing nhất quán và hỗ trợ system
<div className={cx("flex items-center gap-[0.5rem]")}>
  <Avatar src={user.avatarUrl} size={24} />
  <span className={cx("truncate")}>{user.fullName}</span>
</div>
```

**Đúng:**

```tsx
// ✅
<CreatedByInfo id={user.id} name={user.fullName} avatar={user.avatarUrl} />
```
