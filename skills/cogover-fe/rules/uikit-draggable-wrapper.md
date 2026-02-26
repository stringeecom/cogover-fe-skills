---
title: Cách sử dụng component DraggableWrapper và DraggableItem trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến hành vi kéo thả bị lỗi, hướng kéo sai, hoặc mất khả năng kéo khi render bên trong Modal
tags: draggable, drag-and-drop, react-beautiful-dnd, sortable, ui-kit
---

## Cách sử dụng component DraggableWrapper và DraggableItem trong ui-kit

Đường dẫn component: `ui-kit/src/components/DraggableWrapper` và `ui-kit/src/components/DraggableItem`

Exports: `DraggableWrapper`, `DraggableItem`.

Thư viện nền tảng: `react-beautiful-dnd`.

Cấu trúc dữ liệu:

```ts
// DraggableWrapper props
interface DraggableWrapperProps {
  onDragEnd: (sourceIndex: number, destinationIndex?: number) => void;
  children: React.ReactNode;
  direction?: "horizontal" | "vertical"; // mặc định: "vertical"
  className?: string;
}

// DraggableItem props
interface DraggableItemProps {
  children: (dragHandleProps: DraggableProvidedDragHandleProps | null | undefined) => React.ReactNode;
  draggableId: string;
  index: number;
  isDragDisabled?: boolean | undefined;
}
```

---

### RULE-DW-01: Luôn sử dụng `DraggableWrapper` kết hợp với `DraggableItem` — không sử dụng trực tiếp `DragDropContext` và `Droppable` từ `react-beautiful-dnd`

`DraggableWrapper` đã đóng gói `DragDropContext`, `Droppable` và context chia sẻ hướng kéo (`StringeeDraggableContext`). Tự implement lại sẽ thiếu logic xử lý transform theo hướng kéo mà `DraggableItem` cung cấp.

**Sai:**

```tsx
// ❌ Sử dụng trực tiếp react-beautiful-dnd — thiếu xử lý direction context
import { DragDropContext, Droppable, Draggable } from "react-beautiful-dnd";

<DragDropContext onDragEnd={handleDragEnd}>
  <Droppable droppableId="list">
    {(provided) => (
      <div ref={provided.innerRef} {...provided.droppableProps}>
        {items.map((item, index) => (
          <Draggable key={item.id} draggableId={item.id} index={index}>
            {(provided) => <div ref={provided.innerRef} {...provided.draggableProps}>{item.name}</div>}
          </Draggable>
        ))}
        {provided.placeholder}
      </div>
    )}
  </Droppable>
</DragDropContext>
```

**Đúng:**

```tsx
// ✅ Sử dụng DraggableWrapper + DraggableItem từ ui-kit
import { DraggableWrapper, DraggableItem } from "ui-kit";

<DraggableWrapper onDragEnd={(sourceIndex, destinationIndex) => {
  // xử lý sắp xếp lại
}}>
  {items.map((item, index) => (
    <DraggableItem key={item.id} draggableId={item.id} index={index}>
      {(dragHandleProps) => (
        <div {...dragHandleProps}>{item.name}</div>
      )}
    </DraggableItem>
  ))}
</DraggableWrapper>
```

---

### RULE-DW-02: `children` của `DraggableItem` phải là render function nhận `dragHandleProps` — không phải ReactNode thông thường

`DraggableItem` sử dụng render props pattern. `children` là một function nhận `dragHandleProps` và trả về `React.ReactNode`. Phải spread `dragHandleProps` lên phần tử mà người dùng sẽ dùng để kéo.

**Sai:**

```tsx
// ❌ children là ReactNode thông thường — không hoạt động
<DraggableItem draggableId="item-1" index={0}>
  <div>Item 1</div>
</DraggableItem>
```

**Đúng:**

```tsx
// ✅ children là render function nhận dragHandleProps
<DraggableItem draggableId="item-1" index={0}>
  {(dragHandleProps) => (
    <div {...dragHandleProps}>Item 1</div>
  )}
</DraggableItem>
```

---

### RULE-DW-03: Luôn spread `dragHandleProps` lên phần tử dùng làm tay cầm kéo

`dragHandleProps` chứa các sự kiện và thuộc tính ARIA cần thiết để kích hoạt kéo. Không spread sẽ khiến item không thể kéo được. Có thể spread lên một phần tử con cụ thể nếu chỉ muốn kéo bằng icon grip thay vì toàn bộ item.

**Sai:**

```tsx
// ❌ Không spread dragHandleProps — item không thể kéo
<DraggableItem draggableId="item-1" index={0}>
  {() => (
    <div>Item 1</div>
  )}
</DraggableItem>
```

**Đúng:**

```tsx
// ✅ Spread dragHandleProps lên toàn bộ item
<DraggableItem draggableId="item-1" index={0}>
  {(dragHandleProps) => (
    <div {...dragHandleProps}>Item 1</div>
  )}
</DraggableItem>

// ✅ Hoặc spread dragHandleProps lên icon grip để kéo bằng icon
<DraggableItem draggableId="item-1" index={0}>
  {(dragHandleProps) => (
    <div className="flex gap-2">
      <div {...dragHandleProps}>
        <FontAwesomeIcon icon={faGripDotsVertical} />
      </div>
      <span>Item 1</span>
    </div>
  )}
</DraggableItem>
```

---

### RULE-DW-04: `draggableId` phải là duy nhất trong toàn bộ danh sách — không sử dụng `index` làm `draggableId`

`react-beautiful-dnd` sử dụng `draggableId` làm định danh duy nhất. Sử dụng `index` hoặc giá trị trùng lặp sẽ gây lỗi kéo thả hoặc hành vi không dự đoán được.

**Sai:**

```tsx
// ❌ Sử dụng index làm draggableId — gây lỗi khi sắp xếp lại
{items.map((item, index) => (
  <DraggableItem key={index} draggableId={String(index)} index={index}>
    {(dragHandleProps) => <div {...dragHandleProps}>{item.name}</div>}
  </DraggableItem>
))}
```

**Đúng:**

```tsx
// ✅ Sử dụng ID duy nhất từ dữ liệu
{items.map((item, index) => (
  <DraggableItem key={item.id} draggableId={item.id} index={index}>
    {(dragHandleProps) => <div {...dragHandleProps}>{item.name}</div>}
  </DraggableItem>
))}
```

---

### RULE-DW-05: Sử dụng `direction="horizontal"` khi cần kéo thả theo chiều ngang — mặc định là `"vertical"`

Khi không truyền prop `direction`, `DraggableWrapper` mặc định sử dụng `"vertical"`. Khi `direction="horizontal"`, wrapper tự động thêm class `flex` và `DraggableItem` sẽ chỉ cho phép transform theo trục X.

**Sai:**

```tsx
// ❌ Thêm class flex thủ công và không truyền direction — kéo theo chiều dọc trong layout ngang
<DraggableWrapper onDragEnd={handleDragEnd} className="flex">
  {items.map((item, index) => (
    <DraggableItem key={item.id} draggableId={item.id} index={index}>
      {(dragHandleProps) => <div {...dragHandleProps}>{item.name}</div>}
    </DraggableItem>
  ))}
</DraggableWrapper>
```

**Đúng:**

```tsx
// ✅ Truyền direction="horizontal" — wrapper tự thêm class flex và DraggableItem xử lý transform đúng
<DraggableWrapper onDragEnd={handleDragEnd} direction="horizontal">
  {items.map((item, index) => (
    <DraggableItem key={item.id} draggableId={item.id} index={index}>
      {(dragHandleProps) => <div {...dragHandleProps}>{item.name}</div>}
    </DraggableItem>
  ))}
</DraggableWrapper>
```

---

### RULE-DW-06: Xử lý `destinationIndex` có thể là `undefined` trong callback `onDragEnd`

Khi người dùng kéo item ra ngoài vùng droppable rồi thả, `destinationIndex` sẽ là `undefined`. Phải kiểm tra giá trị này trước khi thực hiện sắp xếp lại.

**Sai:**

```tsx
// ❌ Không kiểm tra destinationIndex — lỗi khi thả ra ngoài vùng droppable
<DraggableWrapper
  onDragEnd={(sourceIndex, destinationIndex) => {
    const newItems = arrayMoveImmutable(items, sourceIndex, destinationIndex);
    setItems(newItems);
  }}
>
```

**Đúng:**

```tsx
// ✅ Kiểm tra destinationIndex trước khi sắp xếp lại
<DraggableWrapper
  onDragEnd={(sourceIndex, destinationIndex) => {
    if (destinationIndex === undefined) return;
    const newItems = arrayMoveImmutable(items, sourceIndex, destinationIndex);
    setItems(newItems);
  }}
>
```

---

### RULE-DW-07: Truyền `hasDraggable={true}` vào `Modal` khi `DraggableWrapper` được render bên trong nó

`react-beautiful-dnd` yêu cầu xử lý portal đặc biệt bên trong Modal. Không có `hasDraggable={true}`, các sự kiện kéo bị lớp sự kiện của Modal bắt và items không thể kéo được.

**Sai:**

```tsx
// ❌ DraggableWrapper bên trong Modal không có hasDraggable — items không thể kéo
{isOpen && (
  <Modal title="Sắp xếp danh sách">
    <DraggableWrapper onDragEnd={handleDragEnd}>
      {items.map((item, index) => (
        <DraggableItem key={item.id} draggableId={item.id} index={index}>
          {(dragHandleProps) => <div {...dragHandleProps}>{item.name}</div>}
        </DraggableItem>
      ))}
    </DraggableWrapper>
  </Modal>
)}
```

**Đúng:**

```tsx
// ✅ Truyền hasDraggable={true} để cho phép sự kiện kéo qua Modal
{isOpen && (
  <Modal title="Sắp xếp danh sách" hasDraggable={true}>
    <DraggableWrapper onDragEnd={handleDragEnd}>
      {items.map((item, index) => (
        <DraggableItem key={item.id} draggableId={item.id} index={index}>
          {(dragHandleProps) => <div {...dragHandleProps}>{item.name}</div>}
        </DraggableItem>
      ))}
    </DraggableWrapper>
  </Modal>
)}
```

---

### RULE-DW-08: Sử dụng `isDragDisabled` trên `DraggableItem` khi cần vô hiệu hóa kéo cho item cụ thể — không ẩn `dragHandleProps`

Khi cần vô hiệu hóa kéo cho một item cụ thể (ví dụ: item đang được chỉnh sửa), sử dụng prop `isDragDisabled` thay vì bỏ qua việc spread `dragHandleProps`.

**Sai:**

```tsx
// ❌ Bỏ qua dragHandleProps có điều kiện — giao diện không nhất quán
<DraggableItem draggableId={item.id} index={index}>
  {(dragHandleProps) => (
    <div {...(item.isEditing ? {} : dragHandleProps)}>
      {item.name}
    </div>
  )}
</DraggableItem>
```

**Đúng:**

```tsx
// ✅ Sử dụng isDragDisabled — item vẫn giữ vị trí trong danh sách nhưng không thể kéo
<DraggableItem draggableId={item.id} index={index} isDragDisabled={item.isEditing}>
  {(dragHandleProps) => (
    <div {...dragHandleProps}>
      {item.name}
    </div>
  )}
</DraggableItem>
```
