## Cách sử dụng component Drawer trong ui-kit

Path: `ui-kit/src/components/Drawer`

Props: `onClose`, `onOverlayClick`, `hasOverlay` (default true), `overlayClassName`, `headingClassName`, `width` (default 600), `height` (default "100svh"), `placement` (default "right"), `title`, `helpText`, `closeIcon`, `triggerEscPress` (default true), `className`. Extends `React.HTMLAttributes<HTMLDivElement>` and `React.PropsWithChildren`.

Placements: `"top-left"`, `"top-right"`, `"bottom-left"`, `"bottom-right"`, `"right"` (alias for top-right).

---

### Usage

```tsx
// Drawer has no open prop — conditionally render to show/hide
{isOpen && (
  <Drawer title="Tiêu đề" onClose={() => setIsOpen(false)}>
    <p>Nội dung</p>
  </Drawer>
)}

// onOverlayClick to close on overlay click (separate from onClose)
{isOpen && (
  <Drawer
    title="Tiêu đề"
    onClose={() => setIsOpen(false)}
    onOverlayClick={() => setIsOpen(false)}
  >
    <p>Nội dung</p>
  </Drawer>
)}

// Custom placement and size
<Drawer title="Chat" onClose={onClose} placement="bottom-right" width={400} height="50svh">
  ...
</Drawer>

// Left panel (top-left)
<Drawer title="Panel" onClose={onClose} placement="top-left">
  ...
</Drawer>

// No overlay
<Drawer title="..." onClose={onClose} hasOverlay={false}>
  ...
</Drawer>

// Nested modal — disable Esc while modal is open to prevent both closing
const [isModalOpen, setIsModalOpen] = useState(false);
<Drawer title="..." onClose={onClose} triggerEscPress={!isModalOpen}>
  <Button onClick={() => setIsModalOpen(true)}>Mở Modal</Button>
  {isModalOpen && <Modal title="..." onClose={() => setIsModalOpen(false)}>...</Modal>}
</Drawer>

// Custom header and overlay style
<Drawer title="..." onClose={onClose} headingClassName="bg-background-secondary" overlayClassName="!opacity-80">
  ...
</Drawer>

// With helpText in header
<Drawer title="Tiêu đề" helpText="Mô tả chi tiết" onClose={onClose}>
  ...
</Drawer>
```

---

### DO / DON'T

```tsx
// ❌ No open prop — renders always visible when mounted
<Drawer open={isOpen} title="..." onClose={onClose}>...</Drawer>

// ❌ Don't self-position via className
<Drawer className="!left-0 !top-0" title="..." onClose={onClose}>...</Drawer>

// ❌ Don't override size via className
<Drawer className="!w-[500px]" title="..." onClose={onClose}>...</Drawer>

// ❌ Don't hide overlay via overlayClassName
<Drawer overlayClassName="!hidden" title="..." onClose={onClose}>...</Drawer>

// ❌ Don't build header manually inside children
<Drawer onClose={onClose}>
  <div className="flex justify-between border-b"><h3>Title</h3><button onClick={onClose}>X</button></div>
  <p>Content</p>
</Drawer>

// ✅ Use placement, width, height props; use title for header
<Drawer title="Tiêu đề" onClose={onClose} placement="top-left" width={400}>
  <p>Nội dung</p>
</Drawer>
```
