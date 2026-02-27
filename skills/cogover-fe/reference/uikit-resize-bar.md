## ResizeBar Component (`uikit-resize-bar`)

Đường dẫn: `ui-kit/src/components/ResizeBar`

Props: `placement` ("left"|"right"), `width`, `minWidth`, `maxWidth`, `onResize`, `onResizeStart`, `onResizeEnd`, `disabled`, `className`, `children`.

- `"right"` — panel ở **bên trái** của thanh; kéo phải tăng độ rộng
- `"left"` — panel ở **bên phải** của thanh; kéo trái tăng độ rộng

---

### RULE-RESIZE-01: Luôn truyền state `width` hiện tại vào prop `width`

```tsx
// ❌ Hardcode — resize luôn bắt đầu từ 300 bất kể thực tế
<ResizeBar placement="right" width={300} onResize={setPanelWidth} />

// ✅
const [panelWidth, setPanelWidth] = useState(300);
<ResizeBar placement="right" width={panelWidth} onResize={setPanelWidth} />
```

---

### RULE-RESIZE-02: Cập nhật state qua `onResize` (liên tục), `onResizeEnd` chỉ cho side effects

```tsx
// ❌ Panel chỉ resize sau khi thả — cảm giác lỗi
<ResizeBar placement="right" width={panelWidth} onResizeEnd={(diff) => setPanelWidth(panelWidth + diff)} />

// ✅
<ResizeBar
  placement="right"
  width={panelWidth}
  onResize={(newWidth) => setPanelWidth(newWidth)}
  onResizeEnd={() => saveToServer(panelWidth)}
/>
```

---

### RULE-RESIZE-03: `onResizeEnd` nhận `diffWidth` (delta), không phải độ rộng cuối

```tsx
// ❌ diffWidth là delta, không phải absolute width
<ResizeBar width={panelWidth} onResize={setPanelWidth} onResizeEnd={(diffWidth) => saveToServer(diffWidth)} />

// ✅
<ResizeBar width={panelWidth} onResize={setPanelWidth} onResizeEnd={() => saveToServer(panelWidth)} />
```

---

### RULE-RESIZE-04: Luôn set `minWidth` và `maxWidth`

```tsx
// ❌ Không có giới hạn — panel có thể về 0px hoặc tràn
<ResizeBar placement="right" width={panelWidth} onResize={setPanelWidth} />

// ✅
<ResizeBar placement="right" width={panelWidth} minWidth={200} maxWidth={600} onResize={setPanelWidth} />
```

---

### RULE-RESIZE-05: Chọn `placement` đúng theo vị trí panel

```tsx
// ✅ Panel ở bên trái → placement="right"
<div style={{ display: "flex" }}>
  <div style={{ width: panelWidth }}>Left panel</div>
  <ResizeBar placement="right" width={panelWidth} onResize={setPanelWidth} />
</div>

// ✅ Panel ở bên phải → placement="left"
<div style={{ display: "flex" }}>
  <ResizeBar placement="left" width={panelWidth} onResize={setPanelWidth} />
  <div style={{ width: panelWidth }}>Right panel</div>
</div>
```
