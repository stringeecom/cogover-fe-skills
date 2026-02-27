## Cách sử dụng component DraggableWrapper và DraggableItem trong ui-kit

Path: `ui-kit/src/components/DraggableWrapper` and `ui-kit/src/components/DraggableItem`

Underlying library: `react-beautiful-dnd`

```ts
interface DraggableWrapperProps {
  onDragEnd: (sourceIndex: number, destinationIndex?: number) => void;
  children: React.ReactNode;
  direction?: "horizontal" | "vertical"; // default: "vertical"
  className?: string;
}

interface DraggableItemProps {
  children: (dragHandleProps: DraggableProvidedDragHandleProps | null | undefined) => React.ReactNode;
  draggableId: string; // must be unique across entire list
  index: number;       // must be 0-based consecutive
  isDragDisabled?: boolean;
}
```

---

### Usage

```tsx
import { DraggableWrapper, DraggableItem } from "ui-kit";

// Basic vertical list (default)
<DraggableWrapper
  onDragEnd={(sourceIndex, destinationIndex) => {
    if (destinationIndex === undefined) return; // dropped outside droppable
    const newItems = arrayMoveImmutable(items, sourceIndex, destinationIndex);
    setItems(newItems);
  }}
>
  {items.map((item, index) => (
    <DraggableItem key={item.id} draggableId={item.id} index={index}>
      {(dragHandleProps) => (
        <div className={cx("flex items-center gap-[0.5rem]")}>
          <span {...dragHandleProps}><GripIcon /></span>  {/* grip-only handle */}
          <span>{item.name}</span>
        </div>
      )}
    </DraggableItem>
  ))}
</DraggableWrapper>

// Horizontal layout
<DraggableWrapper onDragEnd={handleDragEnd} direction="horizontal">
  {tabs.map((tab, index) => (
    <DraggableItem key={tab.id} draggableId={tab.id} index={index}>
      {(dragHandleProps) => <div {...dragHandleProps}>{tab.label}</div>}
    </DraggableItem>
  ))}
</DraggableWrapper>

// Inside Modal — must pass hasDraggable={true}
<Modal title="Sắp xếp danh sách" hasDraggable={true}>
  <DraggableWrapper onDragEnd={handleDragEnd}>
    {items.map((item, index) => (
      <DraggableItem key={item.id} draggableId={item.id} index={index} isDragDisabled={item.locked}>
        {(dragHandleProps) => <div {...dragHandleProps}>{item.name}</div>}
      </DraggableItem>
    ))}
  </DraggableWrapper>
</Modal>
```

---

### DO / DON'T

```tsx
// ❌ Don't use react-beautiful-dnd directly — missing direction context
import { DragDropContext, Droppable, Draggable } from "react-beautiful-dnd";

// ❌ children must be render function, not JSX
<DraggableItem draggableId="item-1" index={0}>
  <div>Item 1</div>  // wrong — not a function
</DraggableItem>

// ❌ Don't use index as draggableId — breaks on reorder
<DraggableItem draggableId={String(index)} index={index}>

// ❌ Don't use non-consecutive or original index after filter
{items.filter(i => i.visible).map((item) => (
  <DraggableItem draggableId={item.id} index={item.originalIndex}>  // wrong
))}

// ❌ Don't skip destinationIndex check
onDragEnd={(src, dst) => setItems(arrayMoveImmutable(items, src, dst))}  // dst can be undefined

// ✅ Always use stable unique IDs and map index
{items.map((item, index) => (
  <DraggableItem key={item.id} draggableId={item.id} index={index}>
    {(dragHandleProps) => <div {...dragHandleProps}>{item.name}</div>}
  </DraggableItem>
))}
```
