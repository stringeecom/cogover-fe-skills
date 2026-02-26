---
title: Cách sử dụng component Tag trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến icon xóa không hiển thị, styling variant sai hoặc layout icon bị lỗi
tags: tag, badge, chip, ui-kit
---

## Cách sử dụng component Tag trong ui-kit

Đường dẫn component: `ui-kit/src/components/Tag`

Props: `variant` (contained/outlined/gray), `size` (small/medium), `selected`, `disabled`, `iconLeft`, `iconRight`, `onDelete`, `showDeleteIcon`, `iconClose`, `rounded`, `isFocused`, `onClick`, `onMouseDown`, `onMouseUp`, `className`, `children`.

---

### RULE-TAG-01: Icon xóa yêu cầu cả `onDelete` VÀ `showDeleteIcon=true` VÀ `iconClose` phải truthy

Icon xóa chỉ render khi `!!onDelete && showDeleteIcon && !!iconClose`. Vì `showDeleteIcon` mặc định là `true` và `iconClose` mặc định là icon FontAwesome ×, yêu cầu tối thiểu là cung cấp `onDelete`.

**Sai:**

```tsx
// ❌ Không có onDelete — icon xóa không bao giờ render ngay cả với showDeleteIcon={true}
<Tag showDeleteIcon={true}>Label</Tag>

// ❌ showDeleteIcon={false} ẩn rõ ràng icon mặc dù có onDelete
<Tag onDelete={handleDelete} showDeleteIcon={false}>Label</Tag>
```

**Đúng:**

```tsx
// ✅ Chỉ cần cung cấp onDelete — icon xuất hiện tự động
<Tag onDelete={() => handleDelete(id)}>Label</Tag>

// ✅ Sử dụng showDeleteIcon={false} chỉ khi bạn muốn ẩn icon có điều kiện
<Tag onDelete={handleDelete} showDeleteIcon={canDelete}>Label</Tag>
```

---

### RULE-TAG-02: Sử dụng `variant` để kiểm soát kiểu trực quan — không override màu qua `className`

Mỗi variant (`contained`, `outlined`, `gray`) có các color tokens riêng cho trạng thái mặc định, hover, disabled và selected. Override qua `className` phá vỡ các chuyển đổi trạng thái này.

**Sai:**

```tsx
// ❌ Override background — các trạng thái disabled và selected bị lỗi
<Tag className="bg-gray-100 text-gray-700">Label</Tag>
```

**Đúng:**

```tsx
// ✅ Sử dụng variant phù hợp
<Tag variant="contained">Blue chip</Tag>
<Tag variant="outlined">Outlined chip</Tag>
<Tag variant="gray">Gray chip</Tag>
```

---

### RULE-TAG-03: Sử dụng `selected` để toggle trạng thái active/chosen — không đổi variants

`selected` áp dụng kiểu "chosen" riêng biệt cho mỗi variant (primary-filled cho `contained`, primary-border cho `outlined`, primary-light cho `gray`). Không chuyển đổi giữa các variants để mô phỏng selection.

**Sai:**

```tsx
// ❌ Đổi variant để mô phỏng selection — hover và các trạng thái khác bị lỗi
<Tag variant={isSelected ? "contained" : "outlined"}>Label</Tag>
```

**Đúng:**

```tsx
// ✅ Giữ variant ổn định, toggle selected
<Tag
  variant="outlined"
  selected={isSelected}
  onClick={() => setSelected(!isSelected)}
>
  Label
</Tag>
```

---

### RULE-TAG-04: Sử dụng props `iconLeft` / `iconRight` — không đặt icons như children

Icons được truyền như children được wrap trong span truncation text và sẽ không có sizing 16×16 hoặc spacing đúng. Sử dụng props `iconLeft`/`iconRight` chuyên dụng thay vào đó.

**Sai:**

```tsx
// ❌ Icon trong children — mất sizing và gap
<Tag>
  <MyIcon /> Label
</Tag>
```

**Đúng:**

```tsx
// ✅ Sử dụng iconLeft / iconRight
<Tag iconLeft={<MyIcon />}>Label</Tag>
<Tag iconRight={<ArrowIcon />}>Label</Tag>
```

---

### RULE-TAG-05: Chỉ sử dụng `iconClose` để thay thế icon × mặc định — không dùng nó để ẩn nút xóa

`iconClose` thay thế icon bên trong nút xóa. Set nó thành `null`/`undefined` ẩn nút (vì điều kiện kiểm tra `!!iconClose`), nhưng đây là side effect. Sử dụng `showDeleteIcon={false}` để ẩn rõ ràng nút xóa.

**Sai:**

```tsx
// ❌ Sử dụng iconClose={null} để ẩn nút xóa — ý định không rõ ràng
<Tag onDelete={handleDelete} iconClose={null}>
  Label
</Tag>
```

**Đúng:**

```tsx
// ✅ Icon xóa tùy chỉnh
<Tag onDelete={handleDelete} iconClose={<TrashIcon />}>Label</Tag>

// ✅ Ẩn nút xóa một cách rõ ràng
<Tag onDelete={handleDelete} showDeleteIcon={false}>Label</Tag>
```

---

### RULE-TAG-06: `disabled` làm cho tag không tương tác — không kết hợp với `onClick`

Khi `disabled` được set, `pointer-events: none` được áp dụng cho toàn bộ tag. Bất kỳ handler `onClick` nào sẽ không kích hoạt.

**Sai:**

```tsx
// ❌ onClick sẽ không bao giờ kích hoạt khi disabled
<Tag disabled onClick={() => handleClick()}>
  Label
</Tag>
```

**Đúng:**

```tsx
// ✅ Bảo vệ lệnh gọi onClick trong code, hoặc đơn giản bỏ onClick khi disabled
<Tag disabled>Label</Tag>

// ✅ Đính kèm onClick có điều kiện
<Tag onClick={isDisabled ? undefined : handleClick} disabled={isDisabled}>Label</Tag>
```
