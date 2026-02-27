## Cách sử dụng component Modal trong ui-kit

Đường dẫn: `ui-kit/src/components/Modal`
Footer: `ui-kit/src/components/Modal/ModalFooter`

Props: `title`, `subTitle`, `helpText`, `size` ("s"=480px, "m"=560px default, "lg"=700px, "xl"=960px, "xxl"=1360px, "large"=1770px), `width`, `onClose`, `showHeader` (default true), `hasBackdrop`, `hideOnClickOutside` (default true), `triggerEscPress` (default true), `fixedHeight`, `modalTop`, `placement` ("left"|"right"), `hasDraggable`, `headingIcon`, `closeIcon`, `renderCloseButton`, `renderFooter`, `renderHeadingIcon`, `modalBodyClassName`, `modalBodyStyle`, `modalMainClassName`, `modalHeadingClassName`, `modalOverlayClassName`, `className`.

```tsx
// Modal cơ bản — mount/unmount để mở/đóng (không có prop open)
{isOpen && (
  <Modal
    title="Tiêu đề"
    onClose={onClose}
    size="lg"
    renderFooter={() => (
      <ModalFooter>
        <Button onClick={onClose}>Hủy</Button>
        <Button variant="primary" onClick={handleSave}>Lưu</Button>
      </ModalFooter>
    )}
  >
    <p>Content</p>
  </Modal>
)}

// Modal không có header
<Modal showHeader={false} onClose={onClose}><CustomHeader /><Content /></Modal>

// Modal với DnD (hasDraggable chỉ khi dùng react-beautiful-dnd)
<Modal title="Reorder" onClose={onClose} hasDraggable={true}>
  <DragDropContext onDragEnd={handleDragEnd}>...</DragDropContext>
</Modal>

// Modal chiếm full height (editor, list)
<Modal title="Editor" onClose={onClose} size="xl" fixedHeight={true}><RichTextEditor /></Modal>

// Modal góc màn hình
<Modal title="Chat" onClose={onClose} placement="left">...</Modal>
```

**DO:** Dùng conditional render để mở/đóng. Dùng `renderFooter` + `ModalFooter` cho footer. Dùng `size` cho kích thước chuẩn. Dùng `hasDraggable` chỉ khi có DnD. Dùng `showHeader={false}` để ẩn header. Dùng `placement` cho modal góc màn hình.

**DON'T:** Truyền prop `open`. Đặt nút footer vào `children`. Dùng `width` khi có `size` phù hợp. Truyền `triggerEscPress={true}` hay `hideOnClickOutside={true}` (đã là mặc định). Dùng `fixedHeight` cho form đơn giản. Tự định vị modal qua `className`.
