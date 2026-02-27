## ConfirmModal Component (`uikit-confirm-modal`)

Đường dẫn: `ui-kit/src/components/ConfirmModal`

Props: `title`, `message`, `onConfirm`, `onClose`, `onCancel`, `loading`, `cancelText`, `confirmText`, `size`, `modalFooterProps`.

`onCancel` mặc định là `onClose` nội bộ. Texts mặc định: "Hủy" và "Đồng ý".

---

### RULE-CONFIRM-01: Dùng `ConfirmModal` cho tất cả xác nhận hành động phá hủy

```tsx
// ❌ Modal tùy chỉnh trùng lặp logic có sẵn
<Modal title="Xóa?" onClose={onClose}>
  <p>Bạn có chắc không?</p>
  <div><Button onClick={onClose}>Hủy</Button><Button onClick={onDelete}>Xóa</Button></div>
</Modal>

// ✅
<ConfirmModal
  title="Xóa cuộc trò chuyện"
  message="Bạn có chắc chắn muốn xóa không?"
  onClose={onClose}
  onConfirm={onDelete}
/>
```

---

### RULE-CONFIRM-02: Bỏ qua `onCancel` khi hành vi giống `onClose`

```tsx
// ❌ Thừa — onCancel đã mặc định là onClose
<ConfirmModal title="Xóa" message="..." onClose={onClose} onCancel={onClose} onConfirm={onDelete} />

// ✅
<ConfirmModal title="Xóa" message="..." onClose={onClose} onConfirm={onDelete} />

// ✅ Chỉ dùng onCancel khi hành vi khác
<ConfirmModal title="Hủy chỉnh sửa" message="..." onClose={onClose} onCancel={handleCancelWithReset} onConfirm={handleDiscard} />
```

---

### RULE-CONFIRM-03: Dùng prop `loading` — không tự tạo disabled + spinner

```tsx
// ❌
<ConfirmModal title="Xóa" message="..." onClose={onClose}
  onConfirm={<Button disabled={isDeleting}>{isDeleting ? <Spinner /> : "Xóa"}</Button>}
/>

// ✅
<ConfirmModal title="Xóa" message="..." loading={isDeleting} onClose={onClose} onConfirm={handleDelete} />
```

---

### RULE-CONFIRM-04: `message` chấp nhận `ReactNode` cho nội dung phong phú

```tsx
// ✅
<ConfirmModal
  title="Xóa file"
  message={<span>Bạn có muốn xóa file <strong>{fileName}</strong> không? Hành động này không thể hoàn tác.</span>}
  onClose={onClose}
  onConfirm={handleDelete}
/>
```

---

### RULE-CONFIRM-05: Tùy chỉnh labels nút cho hành động phá hủy rõ ràng

```tsx
// ❌ "Đồng ý" mơ hồ cho hành động phá hủy
<ConfirmModal title="Rời nhóm" message="..." onClose={onClose} onConfirm={handleLeave} />

// ✅
<ConfirmModal
  title="Rời nhóm"
  message="Bạn sẽ không còn nhận được tin nhắn từ nhóm này."
  cancelText="Ở lại"
  confirmText="Rời nhóm"
  onClose={onClose}
  onConfirm={handleLeave}
/>
```
