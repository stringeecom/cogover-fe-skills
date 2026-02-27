## Tag Component (`uikit-tag`)

Đường dẫn: `ui-kit/src/components/Tag`

Props: `variant` ("contained"|"outlined"|"gray"), `size` ("small"|"medium"), `selected`, `disabled`, `iconLeft`, `iconRight`, `onDelete`, `showDeleteIcon` (mặc định `true`), `iconClose` (mặc định icon ×), `rounded`, `isFocused`, `onClick`, `onMouseDown`, `onMouseUp`, `className`, `children`.

---

### RULE-TAG-01: Icon xóa chỉ render khi có `onDelete` — `showDeleteIcon` và `iconClose` đã có mặc định

```tsx
// ❌ Không có onDelete — icon xóa không bao giờ render
<Tag showDeleteIcon={true}>Label</Tag>

// ✅ Chỉ cần cung cấp onDelete — icon xuất hiện tự động
<Tag onDelete={() => handleDelete(id)}>Label</Tag>

// ✅ Ẩn icon có điều kiện
<Tag onDelete={handleDelete} showDeleteIcon={canDelete}>Label</Tag>
```

---

### RULE-TAG-02: Dùng `variant` — không override màu qua `className`

```tsx
// ❌ Trạng thái disabled/selected bị lỗi
<Tag className="bg-gray-100 text-gray-700">Label</Tag>

// ✅
<Tag variant="contained">Blue chip</Tag>
<Tag variant="outlined">Outlined chip</Tag>
<Tag variant="gray">Gray chip</Tag>
```

---

### RULE-TAG-03: Dùng `selected` để toggle trạng thái — không đổi variants

```tsx
// ❌ Đổi variant để mô phỏng selection — hover/states bị lỗi
<Tag variant={isSelected ? "contained" : "outlined"}>Label</Tag>

// ✅
<Tag variant="outlined" selected={isSelected} onClick={() => setSelected(!isSelected)}>Label</Tag>
```

---

### RULE-TAG-04: Dùng `iconLeft`/`iconRight` — không đặt icons trong children

```tsx
// ❌ Mất sizing 16×16 và spacing đúng
<Tag><MyIcon /> Label</Tag>

// ✅
<Tag iconLeft={<MyIcon />}>Label</Tag>
<Tag iconRight={<ArrowIcon />}>Label</Tag>
```

---

### RULE-TAG-05: `disabled` vô hiệu hóa toàn bộ tag — không kết hợp với `onClick`

```tsx
// ❌ onClick sẽ không bao giờ kích hoạt khi disabled
<Tag disabled onClick={() => handleClick()}>Label</Tag>

// ✅
<Tag disabled>Label</Tag>
<Tag onClick={isDisabled ? undefined : handleClick} disabled={isDisabled}>Label</Tag>
```
