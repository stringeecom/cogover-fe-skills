---
title: Cách sử dụng Accordion Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến animation không hoạt động, trạng thái mở/đóng không nhất quán, và layout bị vỡ
tags: accordion, collapse, expand, toggle, ui-kit
---

## Cách sử dụng Accordion Component của ui-kit

Đường dẫn component: `ui-kit/src/components/Accordion`

Props: `title`, `children`, `className`, `headerClassName`, `defaultOpen`.

---

### RULE-ACCORDION-01: Accordion chỉ hỗ trợ uncontrolled mode

Accordion không có prop `isOpen`/`open`/`collapsed` hay `onChange`/`onToggle`. Component tự quản lý internal state thông qua `defaultOpen` để set trạng thái ban đầu. Không cố gắng control trạng thái mở/đóng từ bên ngoài.

**Sai:**
```tsx
// ❌ Accordion không chấp nhận prop open hoặc onChange
const [isOpen, setIsOpen] = useState(false);
<Accordion title="Section" open={isOpen} onChange={setIsOpen}>
  <Content />
</Accordion>
```

**Đúng:**
```tsx
// ✅ Uncontrolled — chỉ set trạng thái ban đầu qua defaultOpen
<Accordion title="Section" defaultOpen={true}>
  <Content />
</Accordion>
```

> Nếu cần controlled mode (external state quản lý mở/đóng), hãy sử dụng component `CollapsibleContainer` thay thế.

---

### RULE-ACCORDION-02: Sử dụng `defaultOpen` thay vì tự quản lý initial state

Không tự conditionally render children hay wrap thêm logic để mở Accordion mặc định. Sử dụng prop `defaultOpen` có sẵn.

**Sai:**
```tsx
// ❌ Tự quản lý visibility — bypass animation logic của component
const [show, setShow] = useState(true);
<Accordion title="FAQ">
  {show && <Answer />}
</Accordion>
```

**Đúng:**
```tsx
// ✅ defaultOpen xử lý trạng thái ban đầu với animation đúng
<Accordion title="FAQ" defaultOpen={true}>
  <Answer />
</Accordion>
```

---

### RULE-ACCORDION-03: Không truyền rõ ràng `defaultOpen={false}` — đó là giá trị mặc định

`defaultOpen` mặc định là `false`. Chỉ truyền khi muốn Accordion mở sẵn ban đầu.

**Sai:**
```tsx
// ❌ Thừa — defaultOpen đã mặc định là false
<Accordion title="Details" defaultOpen={false}>
  <Content />
</Accordion>
```

**Đúng:**
```tsx
// ✅ Bỏ qua khi dùng giá trị mặc định
<Accordion title="Details">
  <Content />
</Accordion>

// ✅ Chỉ truyền khi muốn mở sẵn
<Accordion title="Details" defaultOpen={true}>
  <Content />
</Accordion>
```

---

### RULE-ACCORDION-04: Truyền `title` dạng ReactNode cho header phong phú

`title` chấp nhận cả `string` và `ReactNode`. Sử dụng ReactNode khi cần icon, badge, hoặc styled text trong header.

```tsx
// ✅ Title là string đơn giản
<Accordion title="Thông tin chi tiết">
  <Content />
</Accordion>

// ✅ Title là ReactNode với icon và badge
<Accordion
  title={
    <div className={cx("flex items-center gap-[0.5rem]")}>
      <InfoIcon />
      <span>Thông tin chi tiết</span>
      <Badge count={5} />
    </div>
  }
>
  <Content />
</Accordion>
```

---

### RULE-ACCORDION-05: Sử dụng `headerClassName` để style header, không override bằng children wrapper

Accordion cung cấp `headerClassName` để tuỳ chỉnh style cho phần header (padding, background, border, v.v.). Không tự wrap thêm div bên ngoài để style header.

**Sai:**
```tsx
// ❌ Wrap thêm div — phá vỡ click handler và layout của header
<Accordion title="Section">
  <div className={cx("p-[1rem] bg-gray-100")}>
    {/* Đây là content, không phải header */}
  </div>
</Accordion>
```

**Đúng:**
```tsx
// ✅ Sử dụng headerClassName để style header
<Accordion
  title="Section"
  headerClassName={cx("p-[1rem]", "bg-gray-100", "rounded")}
>
  <Content />
</Accordion>
```

---

### RULE-ACCORDION-06: Sử dụng `className` để style container ngoài cùng

`className` được áp dụng cho div wrapper ngoài cùng của Accordion. Sử dụng prop này cho margin, border, background của toàn bộ accordion.

```tsx
// ✅ Style container với className
<Accordion
  title="Settings"
  className={cx("mb-[1rem]", "rounded border border-gray-200")}
>
  <SettingsPanel />
</Accordion>
```

---

### RULE-ACCORDION-07: Không tự tạo animation collapse/expand cho children

Accordion đã có sẵn height animation (300ms ease-in-out) cho phần content. Không thêm CSS transition hoặc animation thủ công cho children.

**Sai:**
```tsx
// ❌ Thêm animation thủ công — conflict với built-in animation
<Accordion title="Section">
  <div className={cx("transition-all duration-300")}>
    <Content />
  </div>
</Accordion>
```

**Đúng:**
```tsx
// ✅ Để Accordion xử lý animation
<Accordion title="Section">
  <Content />
</Accordion>
```
