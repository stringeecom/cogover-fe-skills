## Cách sử dụng component StringeeDndContext trong ui-kit

Path: `ui-kit/src/components/StringeeDndContext`

Exports: `StringeeDndContext`, `DndContainer`, `useDndHandle`.

```ts
type StringeeDndItemType<T> = {
  dndId: string;               // unique across ALL containers
  droppableContainerIds?: string[]; // allowed drop targets (undefined = any)
  data: T;
};
type Items<T> = Record<string, StringeeDndItemType<T>[]>; // containerID → items
```

---

### Usage

```tsx
// useDndHandle manages all state and handlers
const { items, activeItem, onDragStart, onDragOver, onDragEnd } =
  useDndHandle<Task>({
    initItems: {
      todo: [{ dndId: "task-a", data: { name: "Task A" } }],
      done: [{ dndId: "task-b", data: { name: "Task B" } }],
    },
    onDragEnd: ({ overContainerId, activeContainerIdStart, activeItem }) => {
      if (activeContainerIdStart !== overContainerId) {
        api.moveTask(activeItem.data.id, overContainerId);
      } else {
        api.reorderTasks(overContainerId, items[overContainerId].map(i => i.data.id));
      }
    },
  });

<StringeeDndContext
  items={items}
  activeItem={activeItem}
  onDragStart={onDragStart}
  onDragOver={onDragOver}
  onDragEnd={onDragEnd}
  renderItem={({ item, listeners, attributes, isDragging }) => (
    // spread BOTH listeners AND attributes on drag handle
    <div {...listeners} {...attributes} style={{ opacity: isDragging ? 0.4 : 1 }}>
      {item.data.name}
    </div>
  )}
  renderOverlay={({ item }) => (
    // NO listeners/attributes on overlay — just visual ghost
    <div className="shadow-lg opacity-90">{item.data.name}</div>
  )}
>
  <DndContainer id="todo" />  {/* id must exactly match key in items */}
  <DndContainer id="done" />
</StringeeDndContext>

// Palette/toolbox pattern — rootContainerIds clones instead of moves
const { items } = useDndHandle({
  initItems: {
    palette: [
      { dndId: "tpl-text", droppableContainerIds: ["canvas"], data: { type: "text" } },
    ],
    canvas: [],
  },
  rootContainerIds: ["palette"],
});

// In Modal — hasDraggable required
<Modal title="Reorder" hasDraggable={true}>
  <StringeeDndContext ...>
    <DndContainer id="list" />
  </StringeeDndContext>
</Modal>
```

---

### DO / DON'T

```tsx
// ❌ Don't manage state manually — use useDndHandle
const [items, setItems] = useState(initItems);
const [activeItem, setActiveItem] = useState(null);
// ... manual onDragEnd implementation

// ❌ dndId must be unique across ALL containers, not just one
const initItems = {
  todo: [{ dndId: "1", data: { name: "Task A" } }],
  done: [{ dndId: "1", data: { name: "Task B" } }], // conflict!
};

// ❌ DndContainer id must exactly match items key
const initItems = { "todo": [...] };
<DndContainer id="TodoList" />  // "TodoList" ≠ "todo" → empty container

// ❌ Don't spread listeners/attributes in renderOverlay
renderOverlay={({ item, listeners, attributes }) => <div {...listeners} {...attributes}>...</div>}

// ✅ Overlay is visual only
renderOverlay={({ item }) => <div>{item.data.name}</div>}
```
