---
title: Cách sử dụng CollapsibleContainer Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến uncontrolled state conflicts, unnecessary remounts, và toggle behavior bị hỏng
tags: collapsible, collapse, accordion, toggle, ui-kit
---

## Cách sử dụng CollapsibleContainer Component của ui-kit

Đường dẫn component: `ui-kit/src/components/CollapsibleContainer`

Props: `title`, `children`, `collapsed`, `onToggleCollapse`, `unmountOnHide`, `className`.

### RULE-COLLAPSIBLE-01: Sử dụng uncontrolled mode khi không cần external state

Khi parent không cần track collapsed state, bỏ qua `collapsed` và `onToggleCollapse`. Component tự quản lý internal state của nó.

**Sai:**
```tsx
// ❌ Quản lý external state không cần thiết
const [isCollapsed, setIsCollapsed] = useState(false);
<CollapsibleContainer
  collapsed={isCollapsed}
  onToggleCollapse={() => setIsCollapsed(prev => !prev)}
  title="Details"
>
  <p>Content</p>
</CollapsibleContainer>
```

**Đúng:**
```tsx
// ✅ Uncontrolled — internal state xử lý toggling
<CollapsibleContainer title="Details">
  <p>Content</p>
</CollapsibleContainer>
```

### RULE-COLLAPSIBLE-02: Luôn pair `collapsed` với `onToggleCollapse` trong controlled mode

Khi truyền `collapsed` như controlled prop, internal state bị bypass. Nếu không có `onToggleCollapse`, click header sẽ không có hiệu ứng.

**Sai:**
```tsx
// ❌ collapsed là controlled nhưng thiếu toggle — component bị đóng băng
<CollapsibleContainer collapsed={false} title="Settings">
  <p>Content</p>
</CollapsibleContainer>
```

**Đúng:**
```tsx
// ✅ Controlled mode yêu cầu cả hai props
const [isCollapsed, setIsCollapsed] = useState(false);

<CollapsibleContainer
  title="Settings"
  collapsed={isCollapsed}
  onToggleCollapse={() => setIsCollapsed(prev => !prev)}
>
  <p>Content</p>
</CollapsibleContainer>
```

### RULE-COLLAPSIBLE-03: Sử dụng `unmountOnHide={false}` để preserve children state

Mặc định (`unmountOnHide={true}`), children bị unmount khi collapsed — reset tất cả internal state (forms, scroll position, v.v.). Set `unmountOnHide={false}` khi state của children phải được preserve.

**Sai:**
```tsx
// ❌ Form state bị mất mỗi khi section được collapsed
<CollapsibleContainer title="Filters">
  <FilterForm />
</CollapsibleContainer>
```

**Đúng:**
```tsx
// ✅ FilterForm state được preserve giữa collapse/expand
<CollapsibleContainer title="Filters" unmountOnHide={false}>
  <FilterForm />
</CollapsibleContainer>
```

### RULE-COLLAPSIBLE-04: Không tự control visibility của children

Không bao giờ conditionally render children dựa trên external collapsed flag. Sử dụng built-in `collapsed` prop của component thay thế.

**Sai:**
```tsx
// ❌ Bypass component logic, gây duplicate state
<CollapsibleContainer title="Section" collapsed={isCollapsed}>
  {!isCollapsed && <Content />}
</CollapsibleContainer>
```

**Đúng:**
```tsx
// ✅ Để component xử lý children visibility
<CollapsibleContainer title="Section" collapsed={isCollapsed} onToggleCollapse={toggle}>
  <Content />
</CollapsibleContainer>
```

### RULE-COLLAPSIBLE-05: Pass `title` as ReactNode cho rich header content

`title` chấp nhận bất kỳ `React.ReactNode`, nên icons, badges, hoặc styled text có thể được truyền trực tiếp.

```tsx
// ✅ Rich title với icon và badge
<CollapsibleContainer
  title={
    <div className={cx("flex items-center gap-[0.5rem]")}>
      <SettingsIcon />
      <span>Advanced Settings</span>
      <Badge count={3} />
    </div>
  }
>
  <SettingsPanel />
</CollapsibleContainer>
```
