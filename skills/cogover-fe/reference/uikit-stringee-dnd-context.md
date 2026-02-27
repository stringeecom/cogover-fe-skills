---
title: Cách sử dụng component StringeeDndContext trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến hành vi kéo bị lỗi, containers rỗng âm thầm, ID kéo trùng lặp hoặc xung đột với overlay listeners
tags: dnd, drag-and-drop, sortable, kanban, ui-kit
---

## Cách sử dụng component StringeeDndContext trong ui-kit

Đường dẫn component: `ui-kit/src/components/StringeeDndContext`

Exports: `StringeeDndContext`, `DndContainer`, `useDndHandle`.

Cấu trúc dữ liệu:

```ts
type StringeeDndItemType<T> = {
  dndId: string; // ID kéo duy nhất trên TẤT CẢ containers
  droppableContainerIds?: string[]; // các target thả được phép (undefined = tất cả containers)
  data: T; // payload item thực tế
};

type Items<T> = Record<string, StringeeDndItemType<T>[]>; // containerID → items
```

---

### RULE-DND-01: Luôn sử dụng `useDndHandle` — không tự quản lý trạng thái kéo và handlers

`useDndHandle` đóng gói tất cả logic di chuyển cross-container (`onDragOver`), theo dõi item đang active, clone root container và `arrayMove` khi reorder. Tự implement lại điều này dễ gây lỗi và thiếu các edge cases.

**Sai:**

```tsx
// ❌ Tự quản lý state items và drag callbacks
const [items, setItems] = useState(initItems);
const [activeItem, setActiveItem] = useState(null);

const handleDragEnd = (event) => {
  // tự gọi arrayMove, tìm containers, v.v.
};

<StringeeDndContext items={items} activeItem={activeItem} onDragEnd={handleDragEnd} ... />
```

**Đúng:**

```tsx
// ✅ useDndHandle xử lý mọi thứ nội bộ
const { items, activeItem, onDragStart, onDragOver, onDragEnd } =
  useDndHandle<Task>({ initItems });

<StringeeDndContext
  items={items}
  activeItem={activeItem}
  onDragStart={onDragStart}
  onDragOver={onDragOver}
  onDragEnd={onDragEnd}
  renderItem={...}
>
  <DndContainer id="task-list" />
</StringeeDndContext>
```

---

### RULE-DND-02: `dndId` phải là duy nhất trên TẤT CẢ containers, không chỉ trong một container

`dnd-kit` sử dụng `dndId` làm định danh toàn cục cho tất cả các phần tử draggable và droppable. Các giá trị `dndId` trùng lặp giữa các containers gây xung đột âm thầm — items có thể snap vào vị trí sai hoặc không kéo được.

**Sai:**

```tsx
// ❌ Cùng dndId "1" xuất hiện ở cả hai containers
const initItems = {
  todo: [{ dndId: "1", data: { name: "Task A" } }],
  done: [{ dndId: "1", data: { name: "Task B" } }], // xung đột
};
```

**Đúng:**

```tsx
// ✅ dndId là duy nhất toàn cục — sử dụng uuid hoặc ID có namespace
const initItems = {
  todo: [{ dndId: "task-a", data: { name: "Task A" } }],
  done: [{ dndId: "task-b", data: { name: "Task B" } }],
};
```

---

### RULE-DND-03: Luôn spread cả `listeners` VÀ `attributes` trên phần tử drag handle

`listeners` đính kèm các sự kiện pointer/mouse kích hoạt kéo. `attributes` cung cấp ARIA roles và khả năng truy cập bàn phím. Bỏ qua một trong hai sẽ phá vỡ khởi tạo kéo hoặc hỗ trợ bàn phím.

**Sai:**

```tsx
// ❌ Thiếu attributes — navigation bàn phím bị lỗi
renderItem={({ item, listeners }) => (
  <div {...listeners}>{item.data.name}</div>
)}

// ❌ Thiếu listeners — item không thể kéo được
renderItem={({ item, attributes }) => (
  <div {...attributes}>{item.data.name}</div>
)}
```

**Đúng:**

```tsx
// ✅ Spread cả hai trên cùng phần tử drag handle
renderItem={({ item, listeners, attributes, isDragging }) => (
  <div
    {...listeners}
    {...attributes}
    style={{ opacity: isDragging ? 0.4 : 1 }}
  >
    {item.data.name}
  </div>
)}
```

---

### RULE-DND-04: Không spread `listeners` hoặc `attributes` trong `renderOverlay`

`renderOverlay` render phần tử ghost theo cursor trong khi kéo. Nó không phải là phần tử draggable — spread `listeners` hoặc `attributes` trên nó gây xung đột sự kiện với item draggable thực.

**Sai:**

```tsx
// ❌ Overlay không phải draggable — listeners gây xung đột
renderOverlay={({ item, listeners, attributes }) => (
  <div {...listeners} {...attributes}>{item.data.name}</div>
)}
```

**Đúng:**

```tsx
// ✅ Overlay chỉ nhận { item } — không có listeners hoặc attributes
renderOverlay={({ item }) => (
  <div className="shadow-lg opacity-90">{item.data.name}</div>
)}
```

---

### RULE-DND-05: `DndContainer` `id` phải khớp chính xác với một key trong map `items`

`DndContainer` đọc danh sách item của nó từ context sử dụng `id` của nó làm key (`items[id] ?? []`). Một sự không khớp âm thầm render một container rỗng không có lỗi.

**Sai:**

```tsx
// ❌ "TodoList" ≠ "todo" — container render rỗng
const initItems = { "todo": [...] };

<DndContainer id="TodoList" />
```

**Đúng:**

```tsx
// ✅ id khớp chính xác
const initItems = { "todo": [...], "done": [...] };

<DndContainer id="todo" />
<DndContainer id="done" />
```

---

### RULE-DND-06: Hiểu `rootContainerIds` — items kéo từ root containers được clone, không di chuyển

Khi `rootContainerIds` được set, kéo một item ra khỏi root container gán một `uuid` mới cho item gốc tại chỗ, nên entry palette vẫn còn. Clone (với `dndId` gốc) là cái di chuyển đến container target. Không mong đợi các items palette biến mất sau khi được kéo.

**Sai:**

```tsx
// ❌ Mong đợi item palette bị xóa sau khi kéo nó vào canvas
const { items } = useDndHandle({
  initItems: {
    palette: [{ dndId: "tpl-1", data: { type: "text" } }],
    canvas: [],
  },
  rootContainerIds: ["palette"],
});
// Sau khi kéo "tpl-1" vào canvas: palette vẫn có một item (dndId mới), canvas có "tpl-1"
```

**Đúng:**

```tsx
// ✅ Palette luôn giữ lại items của nó — sử dụng rootContainerIds cho patterns toolbox/template
const { items } = useDndHandle({
  initItems: {
    palette: [
      {
        dndId: "tpl-text",
        droppableContainerIds: ["canvas"],
        data: { type: "text" },
      },
      {
        dndId: "tpl-image",
        droppableContainerIds: ["canvas"],
        data: { type: "image" },
      },
    ],
    canvas: [],
  },
  rootContainerIds: ["palette"],
});
```

---

### RULE-DND-07: Sử dụng `droppableContainerIds` trên items để hạn chế containers nào chúng có thể được thả vào

Nếu `droppableContainerIds` là `undefined`, item có thể được thả vào bất kỳ container nào. Set nó rõ ràng khi items chỉ nên được chấp nhận bởi các containers cụ thể (ví dụ: palette items chỉ nên đi vào canvas, không quay lại palette).

**Sai:**

```tsx
// ❌ Không có hạn chế — palette items có thể được thả lại vào chính palette
{ dndId: "tpl-1", data: { type: "text" } }
```

**Đúng:**

```tsx
// ✅ Hạn chế drop chỉ vào canvas
{ dndId: "tpl-1", droppableContainerIds: ["canvas"], data: { type: "text" } }
```

---

### RULE-DND-08: Sử dụng callback `onDragEnd` từ `useDndHandle` để persist thay đổi — dựa vào `activeContainerIdStart` để phát hiện di chuyển cross-container

Khi callback `onDragEnd` kích hoạt, state `items` nội bộ đã được cập nhật. Chỉ sử dụng callback này để đồng bộ thay đổi lên server. `activeContainerIdStart` cho bạn biết item bắt nguồn từ đâu (trước khi kéo).

**Đúng:**

```tsx
const { items, ...handlers } = useDndHandle<Task>({
  initItems,
  onDragEnd: ({ overContainerId, activeContainerIdStart, activeItem }) => {
    if (activeContainerIdStart !== overContainerId) {
      // Item di chuyển sang container khác — cập nhật trên server
      api.moveTask(activeItem.data.id, overContainerId);
    } else {
      // Item được sắp xếp lại trong cùng container — persist thứ tự mới
      api.reorderTasks(
        overContainerId,
        items[overContainerId as string].map((i) => i.data.id),
      );
    }
  },
});
```

---

### RULE-DND-09: Truyền `hasDraggable={true}` vào `Modal` khi `StringeeDndContext` được render bên trong nó

`react-beautiful-dnd` và `@dnd-kit` yêu cầu xử lý portal đặc biệt bên trong Modal. Không có `hasDraggable={true}`, các sự kiện kéo bị lớp sự kiện của Modal bắt và items không thể kéo được.

**Sai:**

```tsx
// ❌ DnD bên trong Modal không có hasDraggable — items không thể kéo được
{isOpen && (
  <Modal title="Reorder Items">
    <StringeeDndContext ...>
      <DndContainer id="list" />
    </StringeeDndContext>
  </Modal>
)}
```

**Đúng:**

```tsx
// ✅ Truyền hasDraggable={true} để cho phép sự kiện kéo qua Modal
{isOpen && (
  <Modal title="Reorder Items" hasDraggable={true}>
    <StringeeDndContext ...>
      <DndContainer id="list" />
    </StringeeDndContext>
  </Modal>
)}
```
