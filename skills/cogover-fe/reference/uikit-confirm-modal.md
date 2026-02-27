---
title: Cách sử dụng component ConfirmModal trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến hành vi cancel/close bị lỗi, state thừa và UX không nhất quán cho các hành động phá hủy
tags: confirm, modal, dialog, delete, destructive, ui-kit
---

## Cách sử dụng component ConfirmModal trong ui-kit

Đường dẫn component: `ui-kit/src/components/ConfirmModal`

Props: `title`, `message`, `onConfirm`, `onClose`, `onCancel`, `loading`, `cancelText`, `confirmText`, `size`, `modalFooterProps`.

### RULE-CONFIRM-01: Sử dụng `ConfirmModal` cho tất cả xác nhận hành động phá hủy

KHÔNG tự xây dựng modal xác nhận từ đầu. Luôn sử dụng `ConfirmModal` cho delete, leave, reset và các hành động phá hủy khác.

**Sai:**

```tsx
// ❌ Modal tùy chỉnh trùng lặp logic có sẵn
<Modal title="Xóa?" onClose={onClose}>
  <p>Bạn có chắc không?</p>
  <div>
    <Button onClick={onClose}>Hủy</Button>
    <Button onClick={onDelete}>Xóa</Button>
  </div>
</Modal>
```

**Đúng:**

```tsx
// ✅
<ConfirmModal
  title="Xóa cuộc trò chuyện"
  message="Bạn có chắc chắn muốn xóa không?"
  onClose={onClose}
  onConfirm={onDelete}
/>
```

### RULE-CONFIRM-02: Bỏ qua `onCancel` khi nó có cùng hành vi với `onClose`

`onCancel` mặc định là `onClose` nội bộ. Chỉ truyền `onCancel` khi nút Cancel cần logic khác với đóng modal.

**Sai:**

```tsx
// ❌ Thừa — onCancel đã mặc định là onClose rồi
<ConfirmModal
  title="Xóa"
  message="..."
  onClose={onClose}
  onCancel={onClose}
  onConfirm={onDelete}
/>
```

**Đúng:**

```tsx
// ✅ Bỏ qua onCancel khi hành vi giống với onClose
<ConfirmModal
  title="Xóa"
  message="..."
  onClose={onClose}
  onConfirm={onDelete}
/>
```

```tsx
// ✅ Chỉ cung cấp onCancel khi hành vi khác với onClose
<ConfirmModal
  title="Hủy chỉnh sửa"
  message="Thay đổi của bạn sẽ không được lưu."
  onClose={onClose}
  onCancel={handleCancelWithReset}
  onConfirm={handleDiscard}
/>
```

### RULE-CONFIRM-03: Sử dụng prop `loading` thay vì disabled + spinner thủ công

**Sai:**

```tsx
// ❌ Trùng lặp những gì prop loading đã xử lý
const [isDeleting, setIsDeleting] = useState(false);
<ConfirmModal
  title="Xóa"
  message="..."
  onClose={onClose}
  onConfirm={
    <Button disabled={isDeleting}>{isDeleting ? <Spinner /> : "Xóa"}</Button>
  }
/>;
```

**Đúng:**

```tsx
// ✅ loading xử lý trạng thái disabled và spinner trên nút Confirm
<ConfirmModal
  title="Xóa"
  message="..."
  loading={isDeleting}
  onClose={onClose}
  onConfirm={handleDelete}
/>
```

### RULE-CONFIRM-04: Truyền `message` như ReactNode cho nội dung phong phú

`message` chấp nhận `React.ReactNode`, vì vậy nội dung động với nhấn mạnh hoặc định dạng có thể được truyền trực tiếp.

```tsx
// ✅
<ConfirmModal
  title="Xóa file"
  message={
    <span>
      Bạn có muốn xóa file <strong>{fileName}</strong> không? Hành động này
      không thể hoàn tác.
    </span>
  }
  onClose={onClose}
  onConfirm={handleDelete}
/>
```

### RULE-CONFIRM-05: Tùy chỉnh labels nút khi text mặc định không rõ ràng

Texts mặc định là i18n "Hủy" và "Đồng ý". Với các hành động phá hủy, override bằng động từ cụ thể để làm rõ ý định.

**Sai:**

```tsx
// ❌ "Đồng ý" mơ hồ cho một hành động phá hủy
<ConfirmModal
  title="Rời nhóm"
  message="..."
  onClose={onClose}
  onConfirm={handleLeave}
/>
```

**Đúng:**

```tsx
// ✅ Labels rõ ràng giảm lỗi người dùng
<ConfirmModal
  title="Rời nhóm"
  message="Bạn sẽ không còn nhận được tin nhắn từ nhóm này."
  cancelText="Ở lại"
  confirmText="Rời nhóm"
  onClose={onClose}
  onConfirm={handleLeave}
/>
```
