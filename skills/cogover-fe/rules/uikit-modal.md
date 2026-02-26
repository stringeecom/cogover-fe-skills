---
title: Cách sử dụng component Modal trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến layout bị lỗi, drag-and-drop thất bại, kích thước sai và hành vi modal không nhất quán
tags: modal, dialog, overlay, ui-kit, react-beautiful-dnd
---

## Cách sử dụng component Modal trong ui-kit

Đường dẫn component: `ui-kit/src/components/Modal`
Footer component: `ui-kit/src/components/Modal/ModalFooter`

Props: `title`, `subTitle`, `helpText`, `size`, `width`, `onClose`, `showHeader`, `hasBackdrop`, `hideOnClickOutside`, `triggerEscPress`, `fixedHeight`, `modalTop`, `placement`, `hasDraggable`, `headingIcon`, `closeIcon`, `renderCloseButton`, `renderFooter`, `renderHeadingIcon`, `renderSubTitleLeftIconClose`, `modalBodyClassName`, `modalBodyStyle`, `modalMainClassName`, `modalHeadingClassName`, `modalOverlayClassName`, `modalCloseButtonClassName`, `modalTitleClassName`, `className`.

---

### RULE-MODAL-01: Render có điều kiện Modal để mở/đóng nó

`Modal` không có prop `open`/`visible`. Mount nó để hiển thị, unmount nó để ẩn.

**Sai:**

```tsx
// ❌ Modal không chấp nhận prop open
<Modal open={isOpen} title="..." onClose={onClose}>
  ...
</Modal>
```

**Đúng:**

```tsx
// ✅
{
  isOpen && (
    <Modal title="..." onClose={onClose}>
      ...
    </Modal>
  );
}
```

---

### RULE-MODAL-02: Sử dụng `renderFooter` với `ModalFooter` cho nội dung footer

Không đặt các nút footer vào trong `children`. Sử dụng `renderFooter` với component `ModalFooter` để có padding, border và hành vi sticky đúng.

**Sai:**

```tsx
// ❌ Footer được đặt trong children — không có border, padding sai, không sticky
<Modal title="..." onClose={onClose}>
  <p>Content</p>
  <div className="flex gap-2 mt-4">
    <Button onClick={onClose}>Hủy</Button>
    <Button variant="primary">Lưu</Button>
  </div>
</Modal>
```

**Đúng:**

```tsx
// ✅
<Modal
  title="..."
  onClose={onClose}
  renderFooter={() => (
    <ModalFooter>
      <Button onClick={onClose}>Hủy</Button>
      <Button variant="primary">Lưu</Button>
    </ModalFooter>
  )}
>
  <p>Content</p>
</Modal>
```

---

### RULE-MODAL-03: Chỉ truyền `hasDraggable={true}` khi sử dụng react-beautiful-dnd bên trong Modal

`hasDraggable={true}` chuyển định vị modal từ `position: absolute` sang flexbox centering, điều này cần thiết để `react-beautiful-dnd` hoạt động đúng bên trong Modal. Nếu nội dung modal không sử dụng drag-and-drop, bỏ qua prop này hoàn toàn.

**Sai:**

```tsx
// ❌ hasDraggable được set không cần thiết — gây thay đổi layout vô lý
<Modal title="Settings" onClose={onClose} hasDraggable={true}>
  <SettingsForm />
</Modal>

// ❌ Thiếu hasDraggable — DnD sẽ bị lỗi trong modal
<Modal title="Reorder Items" onClose={onClose}>
  <DragDropContext onDragEnd={handleDragEnd}>
    <Droppable droppableId="list">
      {(provided) => <div ref={provided.innerRef} {...provided.droppableProps}>...</div>}
    </Droppable>
  </DragDropContext>
</Modal>
```

**Đúng:**

```tsx
// ✅ hasDraggable chỉ khi DnD được dùng
<Modal title="Reorder Items" onClose={onClose} hasDraggable={true}>
  <DragDropContext onDragEnd={handleDragEnd}>
    <Droppable droppableId="list">
      {(provided) => <div ref={provided.innerRef} {...provided.droppableProps}>...</div>}
    </Droppable>
  </DragDropContext>
</Modal>

// ✅ Không có hasDraggable khi không có DnD
<Modal title="Settings" onClose={onClose}>
  <SettingsForm />
</Modal>
```

---

### RULE-MODAL-04: Sử dụng `size` cho các độ rộng chuẩn, `width` cho độ rộng tùy chỉnh

Các size có sẵn: `"s"` (480px), `"m"` (560px, mặc định), `"lg"` (700px), `"xl"` (960px), `"xxl"` (1360px), `"large"` (1770px). Chỉ sử dụng prop chuỗi `width` khi không có size định trước nào phù hợp.

**Sai:**

```tsx
// ❌ Width tùy chỉnh khi có size chuẩn phù hợp
<Modal title="..." onClose={onClose} width="700px">
  ...
</Modal>
```

**Đúng:**

```tsx
// ✅ Sử dụng size định trước
<Modal title="..." onClose={onClose} size="lg">
  ...
</Modal>

// ✅ Width tùy chỉnh khi thực sự cần
<Modal title="..." onClose={onClose} width="820px">
  ...
</Modal>
```

---

### RULE-MODAL-05: Sử dụng `showHeader={false}` khi không cần header

Không truyền `title` rỗng để ẩn header một cách trực quan. Sử dụng `showHeader={false}` để xóa hoàn toàn phần header.

**Sai:**

```tsx
// ❌ Title rỗng vẫn render thanh header
<Modal title="" onClose={onClose}>
  <CustomHeader />
  <Content />
</Modal>
```

**Đúng:**

```tsx
// ✅
<Modal showHeader={false} onClose={onClose}>
  <CustomHeader />
  <Content />
</Modal>
```

---

### RULE-MODAL-06: Không truyền rõ ràng các giá trị mặc định cho `triggerEscPress` và `hideOnClickOutside`

Cả hai props đều mặc định là `true`. Chỉ truyền chúng khi bạn cần vô hiệu hóa hành vi.

**Sai:**

```tsx
// ❌ Thừa — các prop này đã mặc định là true rồi
<Modal
  title="..."
  onClose={onClose}
  triggerEscPress={true}
  hideOnClickOutside={true}
>
  ...
</Modal>
```

**Đúng:**

```tsx
// ✅ Bỏ qua khi dùng hành vi mặc định
<Modal title="..." onClose={onClose}>
  ...
</Modal>

// ✅ Chỉ truyền để vô hiệu hóa
<Modal title="..." onClose={onClose} triggerEscPress={false} hideOnClickOutside={false}>
  ...
</Modal>
```

---

### RULE-MODAL-07: Chỉ sử dụng `fixedHeight` khi modal body nên lấp đầy viewport

Mặc định modal body mở rộng để vừa nội dung cho đến max height. Chỉ sử dụng `fixedHeight={true}` khi body nên luôn lấp đầy không gian có sẵn (ví dụ: editor hoặc list full-height).

**Sai:**

```tsx
// ❌ fixedHeight dùng cho form đơn giản — gây khoảng trống không cần thiết
<Modal title="Edit Profile" onClose={onClose} fixedHeight={true}>
  <ProfileForm />
</Modal>
```

**Đúng:**

```tsx
// ✅ fixedHeight cho vùng nội dung full-height
<Modal title="Document Editor" onClose={onClose} size="xl" fixedHeight={true}>
  <RichTextEditor />
</Modal>
```

---

### RULE-MODAL-08: Sử dụng `placement` cho các modal neo góc

Sử dụng `placement="left"` hoặc `placement="right"` khi modal nên xuất hiện ở góc dưới bên trái hoặc góc dưới bên phải của màn hình. Không tự định vị modal bằng các override `className`.

**Sai:**

```tsx
// ❌ Định vị thủ công qua className
<Modal
  title="Chat"
  onClose={onClose}
  className="!top-auto !bottom-10 !left-10 !translate-x-0"
>
  ...
</Modal>
```

**Đúng:**

```tsx
// ✅
<Modal title="Chat" onClose={onClose} placement="left">
  ...
</Modal>
```
