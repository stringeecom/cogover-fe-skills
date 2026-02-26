---
title: Cách sử dụng ToggleCollapseButton Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến hiển thị icon chevron ngược hướng, border-radius sai phía, và variant màu không nhất quán
tags: toggle, collapse, button, chevron, sidebar, ui-kit
---

## Cách sử dụng ToggleCollapseButton Component của ui-kit

Đường dẫn component: `ui-kit/src/components/ToggleCollapseButton`

Props: `isCollapsed`, `variant`, `placement`, `className`. Hỗ trợ tất cả `React.ButtonHTMLAttributes<HTMLButtonElement>`.

### RULE-TCB-01: Luôn truyền `isCollapsed` để đồng bộ hướng icon chevron

`isCollapsed` là prop bắt buộc, quyết định hướng icon chevron hiển thị. Phải đồng bộ giá trị này với state collapsed thực tế của panel/sidebar liền kề.

**Sai:**
```tsx
// ❌ Hardcode isCollapsed — icon không phản ánh trạng thái thực tế
<ToggleCollapseButton isCollapsed={false} onClick={toggleSidebar} />
```

**Đúng:**
```tsx
// ✅ isCollapsed đồng bộ với state thực tế của sidebar
const [isCollapsed, setIsCollapsed] = useState(false);

<ToggleCollapseButton
  isCollapsed={isCollapsed}
  onClick={() => setIsCollapsed(prev => !prev)}
/>
```

### RULE-TCB-02: Chọn `placement` phù hợp với vị trí đặt button

`placement` quyết định hướng border-radius và hướng icon chevron. Mặc định là `"right"`. Sử dụng `"left"` khi button nằm ở cạnh trái của panel cần toggle.

- `placement="right"`: Button nằm ở cạnh phải — khi mở thì `rounded-r`, khi đóng thì `rounded-l`.
- `placement="left"`: Button nằm ở cạnh trái — khi mở thì `rounded-l`, khi đóng thì `rounded-r`.

**Sai:**
```tsx
// ❌ Button ở cạnh trái nhưng dùng placement mặc định "right" — border-radius và icon hướng sai
<div className={cx("flex")}>
  <ToggleCollapseButton isCollapsed={isCollapsed} onClick={toggle} />
  <SidebarContent />
</div>
```

**Đúng:**
```tsx
// ✅ Button ở cạnh trái thì dùng placement="left"
<div className={cx("flex")}>
  <ToggleCollapseButton placement="left" isCollapsed={isCollapsed} onClick={toggle} />
  <SidebarContent />
</div>
```

### RULE-TCB-03: Chọn `variant` phù hợp với ngữ cảnh giao diện

Component hỗ trợ 2 variant: `"gray"` (mặc định) và `"primary"`. Sử dụng `"gray"` cho các panel phụ trợ, `"primary"` khi cần nhấn mạnh hành động toggle.

- `"gray"`: `bg-gray-5`, hover `bg-gray-20`, text `text-gray-70`.
- `"primary"`: `bg-primary-light-80`, hover `bg-primary-light-60`, text `text-primary-main`.

```tsx
// ✅ Variant mặc định cho sidebar phụ trợ
<ToggleCollapseButton isCollapsed={isCollapsed} onClick={toggle} />

// ✅ Variant primary cho panel quan trọng
<ToggleCollapseButton variant="primary" isCollapsed={isCollapsed} onClick={toggle} />
```

### RULE-TCB-04: Không tự render icon chevron bên ngoài component

Component đã xử lý logic hiển thị icon `faChevronLeft`/`faChevronRight` dựa trên `isCollapsed` và `placement`. Không tự thêm icon bên ngoài.

**Sai:**
```tsx
// ❌ Duplicate icon logic — component đã render chevron internally
<div className={cx("flex items-center")}>
  <FontAwesomeIcon icon={isCollapsed ? faChevronLeft : faChevronRight} />
  <ToggleCollapseButton isCollapsed={isCollapsed} onClick={toggle} />
</div>
```

**Đúng:**
```tsx
// ✅ Để component tự xử lý icon
<ToggleCollapseButton isCollapsed={isCollapsed} onClick={toggle} />
```

### RULE-TCB-05: Sử dụng `className` để tuỳ chỉnh vị trí, không override kích thước

Component có kích thước cố định `w-[15px] h-[40px]`. Sử dụng `className` để điều chỉnh vị trí (position, margin) chứ không nên override width/height vì sẽ phá vỡ tỉ lệ icon bên trong.

**Sai:**
```tsx
// ❌ Override kích thước phá vỡ tỉ lệ icon
<ToggleCollapseButton
  isCollapsed={isCollapsed}
  onClick={toggle}
  className={cx("w-[2rem] h-[3rem]")}
/>
```

**Đúng:**
```tsx
// ✅ Chỉ tuỳ chỉnh vị trí
<ToggleCollapseButton
  isCollapsed={isCollapsed}
  onClick={toggle}
  className={cx("absolute top-[50%] -translate-y-[50%]")}
/>
```

## Tham chiếu kích thước

| Thuộc tính | Giá trị |
|------------|---------|
| width | 15px |
| height | 40px |
| icon fontSize | 12px |
