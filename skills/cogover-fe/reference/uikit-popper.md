## Popper Component (`uikit-popper`)

Đường dẫn: `ui-kit/src/components/Popper`

Wrapper xung quanh `react-popper` để xây dựng dropdowns, menus, tooltips tùy chỉnh.

---

### RULE-POPPER-01: Cấu trúc render prop bắt buộc

`children` = trigger, `render` = content.

```tsx
<Popper
  render={({ attributes, ...params }) => (
    <div {...params} {...attributes}><PopperContent /></div>
  )}
>
  {({ ref, onClick }) => (
    <button ref={ref} onClick={onClick}>Trigger</button>
  )}
</Popper>
```

---

### RULE-POPPER-02: Luôn spread `params` trước `attributes` trong `render`

```tsx
// ❌ attributes có thể override params
<div {...attributes} {...params}>

// ✅
<div {...params} {...attributes}>
```

---

### RULE-POPPER-03: Dùng `PopperMenu` + `PopperMenuItem` cho dropdown menus

```tsx
// ✅
render={({ attributes, ...params }) => (
  <PopperMenu {...params} {...attributes}>
    <PopperMenuItem onClick={handleAction1}>Action 1</PopperMenuItem>
    <PopperMenuItem onClick={handleAction2}>Action 2</PopperMenuItem>
  </PopperMenu>
)}
```

---

### RULE-POPPER-04: Dùng `ref` (`PopperRef`) để control từ bên ngoài

```tsx
const popperRef = useRef<PopperRef>(null);
<Popper ref={popperRef} ...>...</Popper>
popperRef.current?.hide();
popperRef.current?.show();
```

---

### RULE-POPPER-05: Controlled mode — dùng `isOpen`, không kết hợp với internal `onClick`

```tsx
// ❌
<Popper isOpen={open}>
  {({ ref, onClick }) => <button ref={ref} onClick={onClick}>...</button>}

// ✅
<Popper isOpen={open}>
  {({ ref }) => <button ref={ref} onClick={() => setOpen(p => !p)}>...</button>}
</Popper>
```

---

### RULE-POPPER-06: Dùng `allowHideFunction`/`allowShowFunction` để conditionally block

```tsx
<Popper allowHideFunction={() => canClose} allowShowFunction={() => canOpen}>
  {(params) => <button {...params}>...</button>}
</Popper>
```

---

### RULE-POPPER-07: Đóng tất cả poppers bằng `closeAllPopper`

```tsx
import { closeAllPopper } from "ui-kit/src/components/Popper/helpers";
closeAllPopper();
closeAllPopper({ exceptIds: [popperRef.current?.popperId] });
```

---

### RULE-POPPER-08: Class `hide-on-click` để auto-close khi click item

```tsx
// ❌
<PopperMenuItem onClick={() => { doAction(); popperRef.current?.hide(); }}>Action</PopperMenuItem>

// ✅
<PopperMenuItem className="hide-on-click" onClick={doAction}>Action</PopperMenuItem>
```

---

### RULE-POPPER-09: `appendTo` để render ra ngoài DOM (portal) khi bị clipped

```tsx
const containerRef = useRef<HTMLDivElement>(null);
<Popper appendTo={containerRef}>...</Popper>
```

---

### RULE-POPPER-10: `isFixed` khi anchor element có `position: fixed`

```tsx
<Popper isFixed placement="bottom-start" offset={[0, 8]}>...</Popper>
```
