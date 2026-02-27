---
title: Cách sử dụng component DraggableItem trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến item không kéo được, mất drag handle, index sai thứ tự hoặc xung đột với hướng kéo
tags: draggable, drag-and-drop, react-beautiful-dnd, sortable, ui-kit
---

## Cách sử dụng component DraggableItem trong ui-kit

Đường dẫn component: `ui-kit/src/components/DraggableItem`

Exports: `DraggableItem`, `DraggableItemProps`.

Component này là wrapper của `Draggable` từ `react-beautiful-dnd`, tự động xử lý transform theo hướng kéo (`vertical` hoặc `horizontal`) dựa trên context từ `DraggableWrapper`. **Phải luôn được sử dụng bên trong `DraggableWrapper`.**

Props:

| Prop | Kiểu | Bắt buộc | Mô tả |
|------|------|----------|-------|
| `children` | `(dragHandleProps: DraggableProvidedDragHandleProps \| null \| undefined) => React.ReactNode` | Có | Render function nhận `dragHandleProps` để gắn lên phần tử drag handle |
| `draggableId` | `string` | Có | ID duy nhất cho item kéo, phải là duy nhất trong toàn bộ danh sách |
| `index` | `number` | Có | Vị trí hiện tại của item trong danh sách |
| `isDragDisabled` | `boolean \| undefined` | Không | Vô hiệu hóa khả năng kéo của item |

---

### RULE-DI-01: Luôn sử dụng `DraggableItem` bên trong `DraggableWrapper`

`DraggableItem` phụ thuộc vào `StringeeDraggableContext` được cung cấp bởi `DraggableWrapper` để xác định hướng kéo (`direction`). Sử dụng `DraggableItem` bên ngoài `DraggableWrapper` sẽ khiến context không có giá trị đúng và transform bị lỗi.

**Sai:**

```tsx
// ❌ DraggableItem nằm ngoài DraggableWrapper — thiếu context direction
<div>
  <DraggableItem draggableId="item-1" index={0}>
    {(dragHandleProps) => <div {...dragHandleProps}>Item 1</div>}
  </DraggableItem>
</div>
```

**Đúng:**

```tsx
// ✅ DraggableItem nằm trong DraggableWrapper — context direction được cung cấp đúng
<DraggableWrapper onDragEnd={handleDragEnd}>
  <DraggableItem draggableId="item-1" index={0}>
    {(dragHandleProps) => <div {...dragHandleProps}>Item 1</div>}
  </DraggableItem>
</DraggableWrapper>
```

---

### RULE-DI-02: `children` là render function — phải spread `dragHandleProps` lên phần tử drag handle

`children` nhận `dragHandleProps` từ `react-beautiful-dnd`. Phải spread object này lên phần tử mà người dùng sẽ dùng để kéo. Không spread sẽ khiến item không thể kéo được.

**Sai:**

```tsx
// ❌ Không spread dragHandleProps — item không thể kéo được
<DraggableItem draggableId="item-1" index={0}>
  {() => <div>Item 1</div>}
</DraggableItem>

// ❌ Truyền children dạng JSX thông thường — sai kiểu
<DraggableItem draggableId="item-1" index={0}>
  <div>Item 1</div>
</DraggableItem>
```

**Đúng:**

```tsx
// ✅ Spread dragHandleProps lên phần tử drag handle
<DraggableItem draggableId="item-1" index={0}>
  {(dragHandleProps) => (
    <div {...dragHandleProps}>Item 1</div>
  )}
</DraggableItem>
```

---

### RULE-DI-03: Khi chỉ muốn kéo bằng một phần tử con (drag handle riêng), chỉ spread `dragHandleProps` lên phần tử đó

Nếu muốn toàn bộ item là drag handle, spread `dragHandleProps` lên phần tử ngoài cùng. Nếu chỉ muốn kéo bằng một icon hoặc nút cụ thể, spread `dragHandleProps` lên phần tử đó thay vì phần tử cha.

**Sai:**

```tsx
// ❌ Spread dragHandleProps lên cả item — không thể click vào nội dung bên trong mà không kích hoạt kéo
<DraggableItem draggableId="item-1" index={0}>
  {(dragHandleProps) => (
    <div {...dragHandleProps}>
      <span><GripIcon /></span>
      <input type="text" />
    </div>
  )}
</DraggableItem>
```

**Đúng:**

```tsx
// ✅ Chỉ spread dragHandleProps lên icon grip — input vẫn hoạt động bình thường
<DraggableItem draggableId="item-1" index={0}>
  {(dragHandleProps) => (
    <div>
      <span {...dragHandleProps}><GripIcon /></span>
      <input type="text" />
    </div>
  )}
</DraggableItem>
```

---

### RULE-DI-04: `draggableId` phải là duy nhất trong toàn bộ danh sách và là chuỗi ổn định

`react-beautiful-dnd` sử dụng `draggableId` để định danh phần tử kéo. Trùng lặp `draggableId` gây xung đột âm thầm. Sử dụng `index` làm `draggableId` sẽ gây lỗi khi danh sách thay đổi.

**Sai:**

```tsx
// ❌ Sử dụng index làm draggableId — lỗi khi thêm/xóa item
{items.map((item, index) => (
  <DraggableItem draggableId={String(index)} index={index}>
    {(dragHandleProps) => <div {...dragHandleProps}>{item.name}</div>}
  </DraggableItem>
))}
```

**Đúng:**

```tsx
// ✅ Sử dụng ID ổn định và duy nhất
{items.map((item, index) => (
  <DraggableItem key={item.id} draggableId={item.id} index={index}>
    {(dragHandleProps) => <div {...dragHandleProps}>{item.name}</div>}
  </DraggableItem>
))}
```

---

### RULE-DI-05: `index` phải là số thứ tự liên tục bắt đầu từ 0

`react-beautiful-dnd` yêu cầu `index` là số nguyên liên tục bắt đầu từ 0 (0, 1, 2, ...). Nếu bỏ qua một giá trị index hoặc sử dụng index không liên tục, hành vi kéo thả sẽ bị lỗi.

**Sai:**

```tsx
// ❌ Lọc items nhưng giữ nguyên index gốc — index không liên tục
{items.filter(item => item.visible).map((item) => (
  <DraggableItem draggableId={item.id} index={item.originalIndex}>
    {(dragHandleProps) => <div {...dragHandleProps}>{item.name}</div>}
  </DraggableItem>
))}
```

**Đúng:**

```tsx
// ✅ Sử dụng index từ map — đảm bảo liên tục từ 0
{items.filter(item => item.visible).map((item, index) => (
  <DraggableItem key={item.id} draggableId={item.id} index={index}>
    {(dragHandleProps) => <div {...dragHandleProps}>{item.name}</div>}
  </DraggableItem>
))}
```

---

### RULE-DI-06: Sử dụng `isDragDisabled` để vô hiệu hóa kéo có điều kiện — không ẩn phần tử

Khi cần ngăn một item bị kéo (ví dụ: item đang chỉnh sửa, item bị khóa), sử dụng prop `isDragDisabled`. Không ẩn phần tử hoặc loại bỏ nó khỏi danh sách vì sẽ phá vỡ thứ tự index.

**Sai:**

```tsx
// ❌ Loại bỏ item bị khóa khỏi danh sách — phá vỡ index
{items.filter(item => !item.locked).map((item, index) => (
  <DraggableItem draggableId={item.id} index={index}>
    {(dragHandleProps) => <div {...dragHandleProps}>{item.name}</div>}
  </DraggableItem>
))}
```

**Đúng:**

```tsx
// ✅ Giữ item trong danh sách nhưng vô hiệu hóa kéo
{items.map((item, index) => (
  <DraggableItem
    key={item.id}
    draggableId={item.id}
    index={index}
    isDragDisabled={item.locked}
  >
    {(dragHandleProps) => (
      <div {...dragHandleProps} style={{ opacity: item.locked ? 0.5 : 1 }}>
        {item.name}
      </div>
    )}
  </DraggableItem>
))}
```

---

### RULE-DI-07: Khi sử dụng `DraggableItem` trong `Modal`, phải truyền `hasDraggable={true}` vào `Modal`

`react-beautiful-dnd` yêu cầu xử lý portal đặc biệt bên trong Modal. Không có `hasDraggable={true}`, các sự kiện kéo bị lớp sự kiện của Modal bắt và items không thể kéo được.

**Sai:**

```tsx
// ❌ DraggableItem trong Modal không có hasDraggable — items không thể kéo
<Modal title="Sắp xếp lại">
  <DraggableWrapper onDragEnd={handleDragEnd}>
    <DraggableItem draggableId="item-1" index={0}>
      {(dragHandleProps) => <div {...dragHandleProps}>Item 1</div>}
    </DraggableItem>
  </DraggableWrapper>
</Modal>
```

**Đúng:**

```tsx
// ✅ Truyền hasDraggable={true} để cho phép sự kiện kéo qua Modal
<Modal title="Sắp xếp lại" hasDraggable={true}>
  <DraggableWrapper onDragEnd={handleDragEnd}>
    <DraggableItem draggableId="item-1" index={0}>
      {(dragHandleProps) => <div {...dragHandleProps}>Item 1</div>}
    </DraggableItem>
  </DraggableWrapper>
</Modal>
```

---

### Ví dụ đầy đủ

```tsx
import { DraggableWrapper, DraggableItem } from "ui-kit";

function SortableList() {
  const [items, setItems] = useState([
    { id: "task-1", name: "Nhiệm vụ A" },
    { id: "task-2", name: "Nhiệm vụ B" },
    { id: "task-3", name: "Nhiệm vụ C" },
  ]);

  const handleDragEnd = (sourceIndex: number, destinationIndex?: number) => {
    if (destinationIndex === undefined) return;

    const newItems = [...items];
    const [removed] = newItems.splice(sourceIndex, 1);
    newItems.splice(destinationIndex, 0, removed);
    setItems(newItems);
  };

  return (
    <DraggableWrapper onDragEnd={handleDragEnd} direction="vertical">
      {items.map((item, index) => (
        <DraggableItem key={item.id} draggableId={item.id} index={index}>
          {(dragHandleProps) => (
            <div className={cx("flex items-center gap-[0.5rem]", "rounded border border-gray-200 p-[0.75rem]")}>
              <span {...dragHandleProps}>
                <GripIcon />
              </span>
              <span>{item.name}</span>
            </div>
          )}
        </DraggableItem>
      ))}
    </DraggableWrapper>
  );
}
```

### Ví dụ kéo ngang

```tsx
<DraggableWrapper onDragEnd={handleDragEnd} direction="horizontal" className="flex gap-[0.5rem]">
  {tabs.map((tab, index) => (
    <DraggableItem key={tab.id} draggableId={tab.id} index={index}>
      {(dragHandleProps) => (
        <div {...dragHandleProps} className={cx("px-[1rem] py-[0.5rem]", "rounded bg-gray-100")}>
          {tab.label}
        </div>
      )}
    </DraggableItem>
  ))}
</DraggableWrapper>
```
