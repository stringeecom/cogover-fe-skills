---
title: Cách sử dụng component ResizeBar trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến hành vi resize bị lỗi, giá trị width cũ và hướng resize sai
tags: resize, drag, panel, sidebar, ui-kit
---

## Cách sử dụng component ResizeBar trong ui-kit

Đường dẫn component: `ui-kit/src/components/ResizeBar`

Props: `placement`, `width`, `minWidth`, `maxWidth`, `onResize`, `onResizeStart`, `onResizeEnd`, `disabled`, `className`, `children`.

---

### RULE-RESIZE-01: Luôn truyền state `width` hiện tại vào prop `width`

`ResizeBar` snapshot prop `width` tại thời điểm bắt đầu kéo để tính kích thước mới. Nếu bạn truyền giá trị cũ hoặc hardcode, việc resize sẽ luôn bắt đầu từ base sai.

**Sai:**

```tsx
// ❌ Hardcode — resize luôn bắt đầu từ 300 bất kể độ rộng thực tế
<ResizeBar placement="right" width={300} onResize={setPanelWidth} />
```

**Đúng:**

```tsx
// ✅ Luôn truyền giá trị state hiện tại
const [panelWidth, setPanelWidth] = useState(300);

<ResizeBar placement="right" width={panelWidth} onResize={setPanelWidth} />;
```

---

### RULE-RESIZE-02: Cập nhật state width qua `onResize`, không phải `onResizeEnd`

`onResize` được gọi liên tục trong quá trình kéo với độ rộng mới đã được clamp. Đây là cái phải điều khiển kích thước panel. `onResizeEnd` chỉ dành cho side effects (ví dụ: persist lên server).

**Sai:**

```tsx
// ❌ Panel chỉ resize sau khi thả chuột — cảm giác bị lỗi trong khi kéo
<ResizeBar
  placement="right"
  width={panelWidth}
  onResizeEnd={(diff) => setPanelWidth(panelWidth + diff)}
/>
```

**Đúng:**

```tsx
// ✅ Cập nhật state liên tục trong khi kéo
<ResizeBar
  placement="right"
  width={panelWidth}
  onResize={(newWidth) => setPanelWidth(newWidth)}
  onResizeEnd={() => saveToServer(panelWidth)}
/>
```

---

### RULE-RESIZE-03: `onResizeEnd` nhận `diffWidth` (delta), không phải độ rộng cuối cùng

Không sử dụng argument của `onResizeEnd` như độ rộng cuối cùng. Sử dụng giá trị state được cập nhật bởi `onResize` thay vào đó.

**Sai:**

```tsx
// ❌ diffWidth là một delta, không phải độ rộng tuyệt đối
<ResizeBar
  placement="right"
  width={panelWidth}
  onResize={setPanelWidth}
  onResizeEnd={(diffWidth) => saveToServer(diffWidth)}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng state hiện tại (đã được cập nhật qua onResize) cho side effects
<ResizeBar
  placement="right"
  width={panelWidth}
  onResize={setPanelWidth}
  onResizeEnd={() => saveToServer(panelWidth)}
/>
```

---

### RULE-RESIZE-04: Chọn `placement` dựa trên phía nào panel nằm

`placement` xác định hướng kéo, không phải vị trí trực quan của thanh.

- `"right"` — panel ở bên **trái** của thanh; kéo sang phải tăng độ rộng
- `"left"` — panel ở bên **phải** của thanh; kéo sang trái tăng độ rộng

**Sai:**

```tsx
// ❌ Panel ở bên trái nhưng placement="left" — kéo sang phải sẽ thu nhỏ panel
<div style={{ display: "flex" }}>
  <div style={{ width: panelWidth }}>Left panel</div>
  <ResizeBar placement="left" width={panelWidth} onResize={setPanelWidth} />
</div>
```

**Đúng:**

```tsx
// ✅ Panel ở bên trái → placement="right"
<div style={{ display: 'flex' }}>
  <div style={{ width: panelWidth }}>Left panel</div>
  <ResizeBar placement="right" width={panelWidth} onResize={setPanelWidth} />
</div>

// ✅ Panel ở bên phải → placement="left"
<div style={{ display: 'flex' }}>
  <ResizeBar placement="left" width={panelWidth} onResize={setPanelWidth} />
  <div style={{ width: panelWidth }}>Right panel</div>
</div>
```

---

### RULE-RESIZE-05: Luôn set `minWidth` và `maxWidth` để ngăn kích thước cực đoan

Không có giới hạn, người dùng có thể kéo panel về 0px hoặc vượt quá viewport. Luôn định nghĩa các giới hạn hợp lý.

**Sai:**

```tsx
// ❌ Không có giới hạn — panel có thể thu gọn hoặc tràn
<ResizeBar placement="right" width={panelWidth} onResize={setPanelWidth} />
```

**Đúng:**

```tsx
// ✅
<ResizeBar
  placement="right"
  width={panelWidth}
  minWidth={200}
  maxWidth={600}
  onResize={setPanelWidth}
/>
```

---

### RULE-RESIZE-06: Không tự implement lại logic resize với `onResizeStart` + `onResizeEnd`

`ResizeBar` xử lý tất cả các sự kiện mouse/pointer/touch nội bộ. Không nhân đôi logic kéo bên ngoài. Chỉ sử dụng `onResizeStart`/`onResizeEnd` cho thay đổi UI state (ví dụ: hiển thị/ẩn overlays) hoặc persistence.

**Sai:**

```tsx
// ❌ Nhân đôi logic nội bộ — dẫn đến xung đột
const handleStart = (e) => {
  startX.current = e.clientX;
  window.addEventListener("mousemove", handleMove);
};

<ResizeBar
  placement="right"
  width={panelWidth}
  onResizeStart={handleStart}
  onResize={setPanelWidth}
/>;
```

**Đúng:**

```tsx
// ✅ Chỉ sử dụng onResizeStart cho side effects
<ResizeBar
  placement="right"
  width={panelWidth}
  onResize={setPanelWidth}
  onResizeStart={() => setIsResizing(true)}
  onResizeEnd={() => {
    setIsResizing(false);
    saveToServer(panelWidth);
  }}
/>
```
