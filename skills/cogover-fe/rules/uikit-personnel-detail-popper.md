---
title: Cách sử dụng PersonnelDetailPopper Component của ui-kit
impact: HIGH
impactDescription: Sử dụng PersonnelDetailPopper sai dẫn đến việc popup thông tin nhân sự không hiển thị, delay hover không đúng, hoặc duplicate logic với Popper + PersonnelDetailCard
tags: personnel, popper, popup, hover, detail-card, ui-kit
---

## Cách sử dụng PersonnelDetailPopper Component của ui-kit

Đường dẫn component: `ui-kit/src/components/PersonnelDetailPopper`

Props kế thừa từ `PersonnelDetailCardProps`: `personnelId`, `isSystemUser`, `isShow`, và tất cả `HTMLAttributes<HTMLDivElement>`.

Props riêng: `children` (bắt buộc), `popperProps` (tuỳ chọn — nhận `Partial<PopperProps>` để tuỳ chỉnh Popper bên trong).

Hành vi mặc định: Khi hover vào `children` sau 500ms sẽ hiện popup chứa `PersonnelDetailCard`, rời chuột sau 200ms sẽ ẩn. Popup hiển thị ở vị trí `bottom-start` và được append vào `document.body`.

---

### RULE-PDP-01: Luôn truyền `personnelId` để popup có dữ liệu hiển thị

`PersonnelDetailCard` bên trong sử dụng `personnelId` để gọi API lấy thông tin nhân sự. Nếu không truyền hoặc truyền `null`, popup sẽ không fetch dữ liệu và hiển thị trống.

**Sai:**

```tsx
// Không truyền personnelId — popup hiển thị trống
<PersonnelDetailPopper>
  <span>{user.name}</span>
</PersonnelDetailPopper>
```

**Đúng:**

```tsx
// Truyền personnelId để popup có dữ liệu
<PersonnelDetailPopper personnelId={user.id}>
  <span>{user.name}</span>
</PersonnelDetailPopper>
```

---

### RULE-PDP-02: Truyền `isSystemUser={true}` khi nhân sự là tài khoản hệ thống

Khi `isSystemUser={true}`, `PersonnelDetailCard` sẽ hiển thị thông tin "Automation Cogover" thay vì gọi API lấy thông tin nhân sự. Sử dụng hằng số `SYSTEM_USER_ID` để kiểm tra.

**Sai:**

```tsx
// Không phân biệt system user — popup vẫn gọi API cho tài khoản hệ thống
<PersonnelDetailPopper personnelId={userId}>
  <span>{userName}</span>
</PersonnelDetailPopper>
```

**Đúng:**

```tsx
import { SYSTEM_USER_ID } from "ui-kit/src/components/DataSelect/RecordMultipleSelect";

<PersonnelDetailPopper
  personnelId={userId}
  isSystemUser={userId === SYSTEM_USER_ID}
>
  <span>{userName}</span>
</PersonnelDetailPopper>
```

---

### RULE-PDP-03: Sử dụng `popperProps` để tuỳ chỉnh Popper, không tự wrap Popper + PersonnelDetailCard

`PersonnelDetailPopper` đã cấu hình sẵn `Popper` với `isShowOnHover`, `delay`, `appendTo`, và `placement`. Khi cần tuỳ chỉnh, truyền qua prop `popperProps` thay vì tự tạo lại.

**Sai:**

```tsx
// Tự wrap Popper + PersonnelDetailCard — duplicate logic, mất cấu hình mặc định
<Popper
  isShowOnHover
  delay={[500, 200]}
  placement="bottom-start"
  appendTo={{ current: document.body }}
  render={({ attributes, ...params }) => (
    <div {...attributes} {...params} className="z-[50] bg-white">
      <PersonnelDetailCard personnelId={user.id} />
    </div>
  )}
>
  {(params) => <div {...params}><span>{user.name}</span></div>}
</Popper>
```

**Đúng:**

```tsx
// Sử dụng PersonnelDetailPopper — đã bao gồm toàn bộ logic
<PersonnelDetailPopper personnelId={user.id}>
  <span>{user.name}</span>
</PersonnelDetailPopper>
```

**Đúng (với tuỳ chỉnh):**

```tsx
// Tuỳ chỉnh placement và offset qua popperProps
<PersonnelDetailPopper
  personnelId={user.id}
  popperProps={{
    placement: "top-start",
    offset: [0, 8],
  }}
>
  <span>{user.name}</span>
</PersonnelDetailPopper>
```

---

### RULE-PDP-04: `children` phải là React element có thể nhận DOM events

`PersonnelDetailPopper` bọc `children` trong một `<div>` bên trong `Popper`, div này nhận `ref` và các event handlers để phát hiện hover. Do đó `children` có thể là bất kỳ `React.ReactNode` nào.

**Sai:**

```tsx
// Truyền null hoặc undefined — không có trigger element
<PersonnelDetailPopper personnelId={user.id}>
  {null}
</PersonnelDetailPopper>
```

**Đúng:**

```tsx
// Truyền element làm trigger
<PersonnelDetailPopper personnelId={user.id}>
  <span className="text-primary-main cursor-pointer">{user.name}</span>
</PersonnelDetailPopper>

// Hoặc truyền component
<PersonnelDetailPopper personnelId={user.id}>
  <Avatar src={user.avatar} alt={user.name} size={32} />
</PersonnelDetailPopper>
```

---

### RULE-PDP-05: Khi dùng trong Avatar, sử dụng prop `id` của Avatar thay vì tự wrap PersonnelDetailPopper

`Avatar` component đã tích hợp sẵn `PersonnelDetailPopper`. Khi truyền prop `id` cho Avatar, popup thông tin nhân sự sẽ tự động được kích hoạt. Không cần tự wrap thêm.

**Sai:**

```tsx
// Tự wrap PersonnelDetailPopper bên ngoài Avatar — duplicate popup
<PersonnelDetailPopper personnelId={user.id}>
  <Avatar src={user.avatar} alt={user.name} size={32} id={user.id} />
</PersonnelDetailPopper>
```

**Đúng:**

```tsx
// Avatar đã tích hợp sẵn PersonnelDetailPopper khi có id
<Avatar
  src={user.avatar}
  alt={user.name}
  size={32}
  id={user.id}
/>

// Tuỳ chỉnh popup qua personnelDetailPopperProps
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

### RULE-PDP-06: Không override các prop mặc định của Popper trừ khi có lý do cụ thể

`PersonnelDetailPopper` đã cấu hình các giá trị mặc định tối ưu cho trải nghiệm hover:
- `delay={[500, 200]}` — 500ms trước khi hiện, 200ms trước khi ẩn
- `isShowOnHover={true}` — hiện khi hover
- `appendTo={{ current: document.body }}` — render popup vào body tránh bị clip
- `placement="bottom-start"` — hiện popup bên dưới, căn trái

Chỉ override qua `popperProps` khi giao diện yêu cầu khác biệt rõ ràng.

**Sai:**

```tsx
// Override delay không cần thiết — mất trải nghiệm hover nhất quán
<PersonnelDetailPopper
  personnelId={user.id}
  popperProps={{
    delay: [0, 0],
    isShowOnHover: false,
  }}
>
  <span>{user.name}</span>
</PersonnelDetailPopper>
```

**Đúng:**

```tsx
// Sử dụng giá trị mặc định — nhất quán với toàn bộ ứng dụng
<PersonnelDetailPopper personnelId={user.id}>
  <span>{user.name}</span>
</PersonnelDetailPopper>
```

---

## Tổng hợp Props

| Prop | Kiểu | Mặc định | Mô tả |
|------|------|----------|-------|
| `children` | `React.ReactNode` | *bắt buộc* | Element trigger hiển thị popup khi hover |
| `personnelId` | `string \| null` | `undefined` | ID nhân sự để fetch thông tin hiển thị trong popup |
| `isSystemUser` | `boolean` | `false` | Hiển thị thông tin "Automation Cogover" thay vì gọi API |
| `isShow` | `boolean` | `true` | Cho phép `PersonnelDetailCard` bên trong render hay không |
| `popperProps` | `Partial<PopperProps>` | `undefined` | Tuỳ chỉnh Popper bên trong (placement, offset, delay...) |

## Cấu hình Popper mặc định

| Thuộc tính | Giá trị | Mô tả |
|------------|---------|-------|
| `delay` | `[500, 200]` | 500ms trước khi hiện, 200ms trước khi ẩn |
| `isShowOnHover` | `true` | Popup hiện khi hover vào trigger |
| `appendTo` | `document.body` | Portal ra body để tránh bị clip bởi overflow |
| `placement` | `bottom-start` | Popup hiện bên dưới, căn trái so với trigger |
