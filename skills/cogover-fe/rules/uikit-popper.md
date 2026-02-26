# Quy tắc Popper Component (`uikit-popper`)

Component `Popper` từ `ui-kit/src/components/Popper` là wrapper xung quanh `react-popper` để xây dựng dropdowns, menus, và custom tooltips.

---

## RULE-POPPER-01 — Cấu trúc render prop bắt buộc

`Popper` sử dụng render prop pattern với hai props bắt buộc: `children` (trigger) và `render` (content).

**Sai:**
```tsx
<Popper>
  <button>Trigger</button>
  <div>Content</div>
</Popper>
```

**Đúng:**
```tsx
<Popper
  render={({ attributes, ...params }) => (
    <div {...params} {...attributes}>
      <PopperContent />
    </div>
  )}
>
  {({ ref, onClick }) => (
    <button ref={ref} onClick={onClick}>Trigger</button>
  )}
</Popper>
```

---

## RULE-POPPER-02 — Thứ tự spread trong render prop

Luôn spread `params` trước `attributes` trong `render` prop. Không đảo ngược thứ tự — `attributes` (popper.js data attributes) không được override `ref`, `style`, hoặc `data-popper-id` từ `params`.

**Sai:**
```tsx
render={({ attributes, ...params }) => (
  <div {...attributes} {...params}>  {/* attributes có thể override params */}
```

**Đúng:**
```tsx
render={({ attributes, ...params }) => (
  <div {...params} {...attributes}>  {/* params trước, attributes sau */}
```

---

## RULE-POPPER-03 — Sử dụng PopperMenu + PopperMenuItem cho dropdown menus

Khi xây dựng standard dropdown menu, sử dụng `PopperMenu` làm container và `PopperMenuItem` cho từng item thay vì custom divs.

**Sai:**
```tsx
render={({ attributes, ...params }) => (
  <div {...params} {...attributes} className="bg-white shadow rounded">
    <div className="px-5 h-10 flex items-center cursor-pointer hover:bg-gray-100">
      Action 1
    </div>
  </div>
)}
```

**Đúng:**
```tsx
render={({ attributes, ...params }) => (
  <PopperMenu {...params} {...attributes}>
    <PopperMenuItem onClick={handleAction1}>Action 1</PopperMenuItem>
    <PopperMenuItem onClick={handleAction2}>Action 2</PopperMenuItem>
  </PopperMenu>
)}
```

---

## RULE-POPPER-04 — Sử dụng `ref` để gọi show/hide từ bên ngoài

Khi popper cần được control từ bên ngoài component, sử dụng `ref` typed là `PopperRef`.

**Sai:**
```tsx
const [isOpen, setIsOpen] = useState(false);
<Popper isOpen={isOpen}>
  {(params) => <button {...params} onClick={() => setIsOpen(false)}>...</button>}
```

**Đúng:**
```tsx
const popperRef = useRef<PopperRef>(null);

<Popper ref={popperRef} ...>
  ...
</Popper>

// Gọi từ nơi khác:
popperRef.current?.hide();
popperRef.current?.show();
```

---

## RULE-POPPER-05 — Sử dụng prop `isOpen` cho controlled mode

Khi open/close state được quản lý bởi parent (controlled), truyền prop `isOpen`. Không kết hợp `isOpen` với internal `onClick` toggle trong `children`.

**Sai:**
```tsx
<Popper isOpen={open}>
  {({ ref, onClick }) => <button ref={ref} onClick={onClick}>...</button>}
  {/* onClick vẫn internally toggle ngay cả trong controlled mode */}
```

**Đúng:**
```tsx
<Popper isOpen={open}>
  {({ ref }) => (
    <button ref={ref} onClick={() => setOpen((prev) => !prev)}>...</button>
  )}
</Popper>
```

---

## RULE-POPPER-06 — Sử dụng `allowHideFunction` / `allowShowFunction` để conditionally block show/hide

Khi business logic cần ngăn popper mở hoặc đóng, sử dụng `allowHideFunction` hoặc `allowShowFunction` thay vì tự quản lý state.

**Sai:**
```tsx
const handleToggle = () => {
  if (!canClose) return;
  setOpen((prev) => !prev);
};
<Popper isOpen={open}>
  {({ ref }) => <button ref={ref} onClick={handleToggle}>...</button>}
```

**Đúng:**
```tsx
<Popper
  allowHideFunction={() => canClose}
  allowShowFunction={() => canOpen}
>
  {(params) => <button {...params}>...</button>}
</Popper>
```

---

## RULE-POPPER-07 — Sử dụng utility `closeAllPopper` để đóng tất cả poppers cùng lúc

Khi tất cả poppers cần được đóng (ví dụ: khi navigate route), sử dụng helper `closeAllPopper` thay vì tự track refs.

```tsx
import { closeAllPopper } from "ui-kit/src/components/Popper/helpers";

// Đóng tất cả
closeAllPopper();

// Đóng tất cả trừ poppers cụ thể
closeAllPopper({ exceptIds: [popperRef.current?.popperId] });
```

---

## RULE-POPPER-08 — Sử dụng class `hide-on-click` để auto-close popper khi click item

Để element bên trong popper tự động đóng nó khi click, thêm class `hide-on-click` thay vì gọi `popperRef.current.hide()` thủ công.

**Sai:**
```tsx
<PopperMenuItem onClick={() => { doAction(); popperRef.current?.hide(); }}>
  Action
</PopperMenuItem>
```

**Đúng:**
```tsx
<PopperMenuItem className="hide-on-click" onClick={doAction}>
  Action
</PopperMenuItem>
```

---

## RULE-POPPER-09 — Sử dụng `appendTo` để render popper ra ngoài DOM tree (portal)

Khi popper bị clipped bởi `overflow: hidden` của parent, sử dụng `appendTo` để render nó vào portal.

```tsx
const containerRef = useRef<HTMLDivElement>(null);

<div ref={containerRef}>
  <Popper appendTo={containerRef}>
    ...
  </Popper>
</div>
```

---

## RULE-POPPER-10 — Sử dụng `isFixed` khi anchor element có `position: fixed`

Khi element mà popper anchor vào có `position: fixed`, hoặc nested bên trong element có `position: fixed`, sử dụng `isFixed={true}`.

```tsx
<Popper isFixed placement="bottom-start" offset={[0, 8]}>
  ...
</Popper>
```
