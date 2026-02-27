## Cách sử dụng component DraggableItem trong ui-kit

Path: `ui-kit/src/components/DraggableItem`

Wrapper of `Draggable` from `react-beautiful-dnd`. Handles transform based on direction context from `DraggableWrapper`. **Must always be used inside `DraggableWrapper`.**

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `children` | `(dragHandleProps: DraggableProvidedDragHandleProps \| null \| undefined) => React.ReactNode` | Yes | Render function receiving dragHandleProps |
| `draggableId` | `string` | Yes | Unique ID across entire list |
| `index` | `number` | Yes | 0-based consecutive position in list |
| `isDragDisabled` | `boolean \| undefined` | No | Disable dragging for this item |

---

### Usage

```tsx
import { DraggableWrapper, DraggableItem } from "ui-kit";

// Basic — spread dragHandleProps on entire item
<DraggableWrapper onDragEnd={handleDragEnd}>
  {items.map((item, index) => (
    <DraggableItem key={item.id} draggableId={item.id} index={index}>
      {(dragHandleProps) => (
        <div {...dragHandleProps}>{item.name}</div>
      )}
    </DraggableItem>
  ))}
</DraggableWrapper>

// Grip-only handle — spread on icon, not container (allows clicking inside)
<DraggableItem draggableId={item.id} index={index}>
  {(dragHandleProps) => (
    <div>
      <span {...dragHandleProps}><GripIcon /></span>
      <input type="text" />
    </div>
  )}
</DraggableItem>

// Disable drag conditionally — keep in list to preserve index order
<DraggableItem draggableId={item.id} index={index} isDragDisabled={item.locked}>
  {(dragHandleProps) => (
    <div {...dragHandleProps} style={{ opacity: item.locked ? 0.5 : 1 }}>
      {item.name}
    </div>
  )}
</DraggableItem>

// In Modal
<Modal title="Sắp xếp lại" hasDraggable={true}>
  <DraggableWrapper onDragEnd={handleDragEnd}>
    <DraggableItem draggableId="item-1" index={0}>
      {(dragHandleProps) => <div {...dragHandleProps}>Item 1</div>}
    </DraggableItem>
  </DraggableWrapper>
</Modal>
```

---

### DO / DON'T

```tsx
// ❌ Must be inside DraggableWrapper
<div><DraggableItem draggableId="item-1" index={0}>...</DraggableItem></div>

// ❌ children must be render function, not JSX
<DraggableItem draggableId="item-1" index={0}>
  <div>Item 1</div>
</DraggableItem>

// ❌ Must spread dragHandleProps — item won't drag without it
<DraggableItem draggableId="item-1" index={0}>
  {() => <div>Item 1</div>}
</DraggableItem>

// ❌ Don't use index as draggableId — breaks when list changes
<DraggableItem draggableId={String(index)} index={index}>

// ❌ Don't use non-consecutive index (e.g. after filter)
{items.filter(i => i.visible).map((item) => (
  <DraggableItem draggableId={item.id} index={item.originalIndex}>
))}

// ✅ Use map index — always 0-based consecutive
{items.filter(i => i.visible).map((item, index) => (
  <DraggableItem key={item.id} draggableId={item.id} index={index}>
    {(dragHandleProps) => <div {...dragHandleProps}>{item.name}</div>}
  </DraggableItem>
))}
```
