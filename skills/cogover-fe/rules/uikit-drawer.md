---
title: Cách sử dụng component Drawer trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến drawer hiển thị sai vị trí, overlay không hoạt động, kích thước sai và hành vi đóng mở không nhất quán
tags: drawer, panel, sidebar, overlay, ui-kit
---

## Cách sử dụng component Drawer trong ui-kit

Đường dẫn component: `ui-kit/src/components/Drawer`

Props: `onClose`, `onOverlayClick`, `hasOverlay`, `overlayClassName`, `headingClassName`, `width`, `height`, `placement`, `title`, `helpText`, `closeIcon`, `triggerEscPress`, `className`. Hỗ trợ tất cả `React.HTMLAttributes<HTMLDivElement>` và `React.PropsWithChildren`.

---

### RULE-DRAWER-01: Render có điều kiện Drawer để mở/đóng nó

`Drawer` không có prop `open`/`visible`. Mount nó để hiển thị, unmount nó để ẩn.

**Sai:**

```tsx
// ❌ Drawer không chấp nhận prop open
<Drawer open={isOpen} title="..." onClose={onClose}>
  ...
</Drawer>
```

**Đúng:**

```tsx
// ✅
{isOpen && (
  <Drawer title="..." onClose={onClose}>
    ...
  </Drawer>
)}
```

---

### RULE-DRAWER-02: Sử dụng `placement` để xác định vị trí hiển thị

Drawer hỗ trợ 4 vị trí: `"top-left"`, `"top-right"` (mặc định `"right"` tương đương `"top-right"`), `"bottom-left"`, `"bottom-right"`. Không tự định vị drawer bằng các override `className`.

**Sai:**

```tsx
// ❌ Định vị thủ công qua className
<Drawer
  title="Panel"
  onClose={onClose}
  className="!left-0 !top-0"
>
  ...
</Drawer>
```

**Đúng:**

```tsx
// ✅ Sử dụng placement
<Drawer title="Panel" onClose={onClose} placement="top-left">
  ...
</Drawer>

// ✅ Drawer ở góc dưới bên phải với chiều cao tùy chỉnh
<Drawer title="Chat" onClose={onClose} placement="bottom-right" height="50svh">
  ...
</Drawer>
```

---

### RULE-DRAWER-03: Sử dụng `width` và `height` để tùy chỉnh kích thước

`width` mặc định là `600` (px), `height` mặc định là `"100svh"`. Cả hai đều nhận giá trị `number` (tính bằng px) hoặc `string` (đơn vị CSS tùy ý). Không ghi đè kích thước bằng `className`.

**Sai:**

```tsx
// ❌ Ghi đè width bằng className
<Drawer title="..." onClose={onClose} className="!w-[500px]">
  ...
</Drawer>
```

**Đúng:**

```tsx
// ✅ Sử dụng prop width (number — tính bằng px)
<Drawer title="..." onClose={onClose} width={500}>
  ...
</Drawer>

// ✅ Sử dụng prop width (string — đơn vị CSS tùy ý)
<Drawer title="..." onClose={onClose} width="50vw">
  ...
</Drawer>

// ✅ Tùy chỉnh cả width và height
<Drawer title="..." onClose={onClose} width={400} height="50svh">
  ...
</Drawer>
```

---

### RULE-DRAWER-04: Truyền `onOverlayClick` để đóng drawer khi nhấn vào overlay

`onClose` xử lý việc đóng drawer qua nút close và phím Esc. `onOverlayClick` xử lý riêng việc nhấn vào overlay. Thông thường cả hai nên gọi cùng một hàm đóng.

**Sai:**

```tsx
// ❌ Thiếu onOverlayClick — nhấn vào overlay không đóng drawer
<Drawer title="..." onClose={() => setShow(false)}>
  ...
</Drawer>
```

**Đúng:**

```tsx
// ✅ Cả hai đều đóng drawer
<Drawer
  title="..."
  onClose={() => setShow(false)}
  onOverlayClick={() => setShow(false)}
>
  ...
</Drawer>
```

---

### RULE-DRAWER-05: Sử dụng `hasOverlay={false}` khi không cần overlay

Mặc định `hasOverlay` là `true`. Khi tắt overlay, drawer sẽ tự động có `shadow-primary` để phân biệt với nội dung phía sau.

**Sai:**

```tsx
// ❌ Ẩn overlay bằng className
<Drawer title="..." onClose={onClose} overlayClassName="!hidden">
  ...
</Drawer>
```

**Đúng:**

```tsx
// ✅ Sử dụng prop hasOverlay
<Drawer title="..." onClose={onClose} hasOverlay={false}>
  ...
</Drawer>
```

---

### RULE-DRAWER-06: Không truyền rõ ràng các giá trị mặc định

Các giá trị mặc định: `placement="right"`, `width={600}`, `height="100svh"`, `hasOverlay={true}`, `triggerEscPress={true}`. Chỉ truyền khi cần thay đổi giá trị mặc định.

**Sai:**

```tsx
// ❌ Thừa — các prop này đã là giá trị mặc định
<Drawer
  title="..."
  onClose={onClose}
  hasOverlay={true}
  width={600}
  height="100svh"
>
  ...
</Drawer>
```

**Đúng:**

```tsx
// ✅ Bỏ qua khi dùng giá trị mặc định
<Drawer title="..." onClose={onClose}>
  ...
</Drawer>

// ✅ Chỉ truyền khi cần thay đổi
<Drawer title="..." onClose={onClose} width={400} placement="top-left">
  ...
</Drawer>
```

---

### RULE-DRAWER-07: Sử dụng `title` để hiển thị header với nút đóng

Khi truyền `title`, drawer tự động render header bao gồm tiêu đề, nút đóng và `helpText` (nếu có). Nếu không truyền `title`, header sẽ không hiển thị.

**Sai:**

```tsx
// ❌ Tự tạo header thủ công bên trong children
<Drawer onClose={onClose}>
  <div className="flex justify-between items-center border-b">
    <h3>Tiêu đề</h3>
    <button onClick={onClose}>X</button>
  </div>
  <p>Nội dung</p>
</Drawer>
```

**Đúng:**

```tsx
// ✅ Sử dụng prop title — header tự động render
<Drawer title="Tiêu đề" onClose={onClose}>
  <p>Nội dung</p>
</Drawer>

// ✅ Với helpText
<Drawer title="Tiêu đề" helpText="Mô tả chi tiết" onClose={onClose}>
  <p>Nội dung</p>
</Drawer>

// ✅ title nhận ReactNode
<Drawer title={<strong>Tiêu đề bôi đậm</strong>} onClose={onClose}>
  <p>Nội dung</p>
</Drawer>
```

---

### RULE-DRAWER-08: Quản lý `triggerEscPress` khi có modal/dialog lồng nhau

Khi bên trong drawer có modal hoặc dialog khác, cần tắt `triggerEscPress` của drawer để tránh đóng cả drawer khi người dùng nhấn Esc để đóng modal bên trong.

**Sai:**

```tsx
// ❌ Nhấn Esc sẽ đóng cả modal lẫn drawer
const [isModalOpen, setIsModalOpen] = useState(false);

<Drawer title="..." onClose={onClose}>
  <Button onClick={() => setIsModalOpen(true)}>Mở Modal</Button>
  {isModalOpen && <Modal title="..." onClose={() => setIsModalOpen(false)}>...</Modal>}
</Drawer>
```

**Đúng:**

```tsx
// ✅ Tắt triggerEscPress khi modal bên trong đang mở
const [isModalOpen, setIsModalOpen] = useState(false);

<Drawer title="..." onClose={onClose} triggerEscPress={!isModalOpen}>
  <Button onClick={() => setIsModalOpen(true)}>Mở Modal</Button>
  {isModalOpen && <Modal title="..." onClose={() => setIsModalOpen(false)}>...</Modal>}
</Drawer>
```

---

### RULE-DRAWER-09: Sử dụng `headingClassName` và `overlayClassName` để tùy chỉnh style

Sử dụng `headingClassName` để tùy chỉnh style phần header và `overlayClassName` để tùy chỉnh style overlay. Không ghi đè style qua CSS selector.

**Sai:**

```tsx
// ❌ Ghi đè style qua CSS global
// .stringee-drawer-overlay { background: rgba(0,0,0,0.8) }
<Drawer title="..." onClose={onClose}>
  ...
</Drawer>
```

**Đúng:**

```tsx
// ✅ Sử dụng overlayClassName
<Drawer title="..." onClose={onClose} overlayClassName="!opacity-80">
  ...
</Drawer>

// ✅ Sử dụng headingClassName
<Drawer title="..." onClose={onClose} headingClassName="bg-background-secondary">
  ...
</Drawer>
```
