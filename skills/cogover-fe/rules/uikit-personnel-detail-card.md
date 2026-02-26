---
title: Cách sử dụng PersonnelDetailCard Component của ui-kit
impact: HIGH
impactDescription: Sử dụng PersonnelDetailCard sai dẫn đến việc tự tạo popup thông tin nhân sự, gọi API trùng lặp, và mất tính nhất quán giao diện
tags: personnel, detail, card, popper, popup, hover, ui-kit
---

## Cách sử dụng PersonnelDetailCard Component của ui-kit

Đường dẫn component: `ui-kit/src/components/PersonnelDetailCard`

Props: `personnelId`, `isSystemUser`, `isShow`. Hỗ trợ tất cả `React.HTMLAttributes<HTMLDivElement>`.

PersonnelDetailCard là component hiển thị thông tin chi tiết nhân sự (avatar, tên, email tài khoản, trạng thái tài khoản, email công việc) trong một card popup. Component tự động gọi API lấy thông tin nhân sự dựa trên `personnelId`.

---

### RULE-PDC-01: Truyền `personnelId` để component tự động gọi API lấy thông tin nhân sự

PersonnelDetailCard sử dụng hook `useQueryPersonnelDetail` để tự động fetch dữ liệu nhân sự. Chỉ cần truyền `personnelId`, không cần tự gọi API rồi truyền dữ liệu vào.

**Sai:**

```tsx
// ❌ Tự gọi API rồi truyền dữ liệu vào — duplicate logic đã có sẵn trong component
const { data: personnel } = useQueryPersonnelDetail({ id: userId });

<div className="p-4 border rounded">
  <Avatar src={personnel.avatar} alt={personnel._full_name} size={36} />
  <p>{personnel._full_name}</p>
  <p>{personnel.account_email}</p>
</div>
```

**Đúng:**

```tsx
// ✅ Chỉ cần truyền personnelId, component tự động fetch và hiển thị
<PersonnelDetailCard personnelId={userId} />
```

---

### RULE-PDC-02: Sử dụng `PersonnelDetailPopper` hoặc prop `personnelDetailCardProps` của Avatar thay vì tự wrap Popper

Khi cần hiển thị PersonnelDetailCard dạng popup khi hover, sử dụng `PersonnelDetailPopper` hoặc truyền qua prop `personnelDetailCardProps` của Avatar. Không tự wrap PersonnelDetailCard trong Popper.

**Sai:**

```tsx
// ❌ Tự wrap Popper — duplicate logic đã có sẵn trong PersonnelDetailPopper
<Popper
  delay={[500, 200]}
  isShowOnHover
  placement="bottom-start"
  render={({ attributes, ...params }) => (
    <div {...attributes} {...params}>
      <PersonnelDetailCard personnelId={user.id} />
    </div>
  )}
>
  {(params) => <div {...params}>{user.name}</div>}
</Popper>
```

**Đúng:**

```tsx
// ✅ Sử dụng PersonnelDetailPopper — đã bao gồm Popper với cấu hình mặc định
import { PersonnelDetailPopper } from "ui-kit";

<PersonnelDetailPopper personnelId={user.id}>
  {user.name}
</PersonnelDetailPopper>
```

```tsx
// ✅ Hoặc sử dụng qua Avatar component với prop id (tự động kích hoạt popup)
<Avatar
  src={user.avatar}
  alt={user.name}
  size={32}
  id={user.id}
/>
```

---

### RULE-PDC-03: Sử dụng `isSystemUser` cho tài khoản hệ thống Automation Cogover

Khi hiển thị thông tin của tài khoản hệ thống (Automation Cogover), truyền `isSystemUser={true}`. Component sẽ hiển thị giao diện đặc biệt với logo Cogover thay vì gọi API lấy thông tin nhân sự.

**Sai:**

```tsx
// ❌ Tự tạo giao diện cho system user — thiếu tính nhất quán
{isSystemUser ? (
  <div className="flex items-center gap-3">
    <img src={logoIcon} alt="logo" className="w-9 h-9" />
    <div>
      <span>Automation Cogover</span>
      <span>cogover.com</span>
    </div>
  </div>
) : (
  <PersonnelDetailCard personnelId={userId} />
)}
```

**Đúng:**

```tsx
// ✅ isSystemUser tự động hiển thị giao diện system user
<PersonnelDetailCard personnelId={userId} isSystemUser={isSystemUser} />
```

---

### RULE-PDC-04: Sử dụng `isShow` để điều khiển hiển thị, không tự wrap điều kiện

PersonnelDetailCard có prop `isShow` (mặc định `true`) để điều khiển việc hiển thị và **đồng thời kiểm soát việc gọi API**. Khi `isShow={false}`, component không render và không gọi API. Không tự viết điều kiện render bên ngoài vì sẽ mất khả năng kiểm soát API call.

**Sai:**

```tsx
// ❌ Tự viết điều kiện — vẫn có thể gọi API dù không hiển thị (nếu component đã mount trước đó)
{shouldShow && <PersonnelDetailCard personnelId={userId} />}
```

**Đúng:**

```tsx
// ✅ isShow kiểm soát cả việc render và việc gọi API
<PersonnelDetailCard personnelId={userId} isShow={shouldShow} />
```

---

### RULE-PDC-05: Truyền `popperProps` qua PersonnelDetailPopper để tuỳ chỉnh vị trí popup

Khi cần tuỳ chỉnh vị trí, delay, hoặc các thuộc tính khác của popup, sử dụng prop `popperProps` của PersonnelDetailPopper. Không tự tạo Popper mới.

**Sai:**

```tsx
// ❌ Tự tạo Popper để tuỳ chỉnh vị trí — mất các cấu hình mặc định
<Popper
  delay={[300, 100]}
  isShowOnHover
  placement="top"
  render={({ attributes, ...params }) => (
    <div {...attributes} {...params}>
      <PersonnelDetailCard personnelId={user.id} />
    </div>
  )}
>
  {(params) => <span {...params}>{user.name}</span>}
</Popper>
```

**Đúng:**

```tsx
// ✅ Tuỳ chỉnh qua popperProps — giữ nguyên các cấu hình mặc định khác
<PersonnelDetailPopper
  personnelId={user.id}
  popperProps={{
    placement: "top",
    delay: [300, 100],
  }}
>
  {user.name}
</PersonnelDetailPopper>
```

---

### RULE-PDC-06: Không tự tạo card thông tin nhân sự bằng HTML/CSS

PersonnelDetailCard đã xử lý: gọi API lấy thông tin nhân sự, hiển thị avatar với AvatarField, trạng thái tài khoản với StatusAccount, email công việc, và nút "Chi tiết" mở trang nhân sự trong tab mới. Không tự tạo lại các tính năng này.

**Sai:**

```tsx
// ❌ Tự tạo card — thiếu tính nhất quán, không có trạng thái tài khoản, thiếu nút chi tiết
const { data: personnel } = useQueryPersonnelDetail({ id: userId });

<div className="p-4 border rounded shadow">
  <div className="flex items-center gap-3">
    <img src={personnel.avatar} className="w-9 h-9 rounded-full" />
    <div>
      <p className="font-bold">{personnel._full_name}</p>
      <p>{personnel.account_email}</p>
    </div>
  </div>
  <p>{personnel.work_email}</p>
  <a href={`/personnel/${personnel.id}`} target="_blank">Chi tiết</a>
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng PersonnelDetailCard để có đầy đủ tính năng và giao diện nhất quán
<PersonnelDetailCard personnelId={userId} />
```

---

## Tham chiếu Props

| Prop | Kiểu | Mặc định | Mô tả |
|------|------|----------|-------|
| `personnelId` | `string \| null` | `undefined` | ID của nhân sự, dùng để gọi API lấy thông tin |
| `isSystemUser` | `boolean` | `false` | Hiển thị giao diện tài khoản hệ thống Automation Cogover |
| `isShow` | `boolean` | `true` | Điều khiển việc hiển thị component và gọi API |

## Các component liên quan

| Component | Mục đích |
|-----------|----------|
| `PersonnelDetailPopper` | Wrap PersonnelDetailCard trong Popper để hiển thị dạng popup khi hover |
| `Avatar` | Có tích hợp sẵn PersonnelDetailPopper qua prop `id` và `personnelDetailCardProps` |
| `AvatarField` | Được sử dụng bên trong PersonnelDetailCard để hiển thị avatar nhân sự |
