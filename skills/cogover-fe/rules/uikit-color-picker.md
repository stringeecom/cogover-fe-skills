---
title: Cách sử dụng component ColorPicker trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến màu không đồng bộ, reset không hoạt động, transparent color hiển thị sai
tags: color-picker, color, hex, rgb, rgba, hsl, ui-kit
---

## Cách sử dụng component ColorPicker trong ui-kit

Đường dẫn component: `ui-kit/src/components/ColorPicker`

Props bắt buộc: `defaultValue`, `onChangeColor`.
Props tùy chọn: `isSelected`, `onClick`, `onClose`, `children`, `width`, `height`, `sizeIconEdit`, `disabled`, `showCurrentColor`, `showReset`, `showTransparent`.

Giá trị mặc định: `width=40`, `height=40`, `sizeIconEdit=16`, `isSelected=false`, `disabled=false`, `showCurrentColor=false`, `showReset=false`, `showTransparent=false`.

Hỗ trợ format: Hex (`#616bc9`), RGB (`rgb(97,107,201)`), RGBA (`rgba(97,107,201,1)`), HSL (`hsl(234,49%,58%)`).

---

### RULE-COLOR-PICKER-01: Truyền `defaultValue` là màu hợp lệ — `onChangeColor` chỉ fire khi màu valid

`defaultValue` là giá trị khởi tạo (hex/rgb/rgba/hsl). `onChangeColor` chỉ được gọi khi người dùng chọn màu hợp lệ (qua `colord.isValid()`). Không dùng `defaultValue` rỗng hoặc invalid.

**Sai:**

```tsx
// ❌ defaultValue rỗng — color picker không có màu khởi tạo
<ColorPicker defaultValue="" onChangeColor={setColor} />

// ❌ defaultValue invalid — onChangeColor không bao giờ fire
<ColorPicker defaultValue="not-a-color" onChangeColor={setColor} />
```

**Đúng:**

```tsx
// ✅ Hex
<ColorPicker defaultValue="#616bc9" onChangeColor={setColor} />

// ✅ RGB
<ColorPicker defaultValue="rgb(97, 107, 201)" onChangeColor={setColor} />

// ✅ RGBA với transparency
<ColorPicker defaultValue="rgba(97, 107, 201, 0.5)" onChangeColor={setColor} />
```

---

### RULE-COLOR-PICKER-02: Sử dụng `showCurrentColor` để hiển thị màu hiện tại làm background nút

Khi `showCurrentColor=true`, nút hiển thị màu hiện tại thay vì icon color picker. Không tự build nút màu bên ngoài.

**Sai:**

```tsx
// ❌ Tự build nút hiển thị màu
<div
  style={{ backgroundColor: color }}
  onClick={() => setShowPicker(true)}
/>
{showPicker && <ColorPicker defaultValue={color} onChangeColor={setColor} />}
```

**Đúng:**

```tsx
// ✅
<ColorPicker
  defaultValue={color}
  onChangeColor={setColor}
  showCurrentColor
/>
```

---

### RULE-COLOR-PICKER-03: Sử dụng `showReset` để hiện nút reset — không tự build reset bên ngoài

Khi `showReset=true`, nút reset xuất hiện khi màu khác `defaultValue`. Click reset trả về màu ban đầu và đúng format.

**Sai:**

```tsx
// ❌ Tự build nút reset bên ngoài — không đồng bộ với state nội bộ
<ColorPicker defaultValue={originalColor} onChangeColor={setColor} />
<Button onClick={() => setColor(originalColor)}>Reset</Button>
```

**Đúng:**

```tsx
// ✅
<ColorPicker
  defaultValue={originalColor}
  onChangeColor={setColor}
  showReset
/>
```

---

### RULE-COLOR-PICKER-04: Sử dụng `showTransparent` khi cần hỗ trợ màu trong suốt

Khi `showTransparent=true`, hiển thị option transparent (checkered pattern). Click transparent set màu thành `rgba(0, 0, 0, 0)` và chuyển format sang RGBA.

**Sai:**

```tsx
// ❌ Tự thêm nút transparent bên ngoài
<div>
  <ColorPicker defaultValue={color} onChangeColor={setColor} />
  <button onClick={() => setColor("rgba(0,0,0,0)")}>Transparent</button>
</div>
```

**Đúng:**

```tsx
// ✅
<ColorPicker
  defaultValue={color}
  onChangeColor={setColor}
  showTransparent
/>
```

---

### RULE-COLOR-PICKER-05: Sử dụng `isSelected` + `onClick` để quản lý selection trong danh sách màu

`isSelected` hiển thị border xanh (active state). `onClick` callback khi click nút màu. `onClose` chỉ fire khi `isSelected=true` và picker đóng với màu hợp lệ. Không tự quản lý border/selection bằng className.

**Sai:**

```tsx
// ❌ Tự quản lý selection bằng className
{colors.map((c, i) => (
  <div className={cx({ "border-info": selectedIndex === i })}>
    <ColorPicker defaultValue={c} onChangeColor={(newColor) => updateColor(i, newColor)} />
  </div>
))}
```

**Đúng:**

```tsx
// ✅
{colors.map((c, i) => (
  <ColorPicker
    key={i}
    defaultValue={c}
    onChangeColor={(newColor) => updateColor(i, newColor)}
    isSelected={selectedIndex === i}
    onClick={() => setSelectedIndex(i)}
    onClose={(finalColor) => saveFinalColor(i, finalColor)}
    showCurrentColor
  />
))}
```

---

### RULE-COLOR-PICKER-06: Sử dụng `children` để tuỳ chỉnh trigger — không wrap ColorPicker trong custom button

`children` nhận render prop giống `Popper` children để tuỳ chỉnh trigger element. Không tự wrap component trong button/div.

**Sai:**

```tsx
// ❌ Wrap trong custom button — double click handler, layout sai
<button onClick={togglePicker}>
  <ColorPicker defaultValue={color} onChangeColor={setColor} />
</button>
```

**Đúng:**

```tsx
// ✅ Custom trigger qua children render prop
<ColorPicker defaultValue={color} onChangeColor={setColor}>
  {(params) => (
    <button {...params} className={cx("px-[1rem] py-[0.5rem]")}>
      Chọn màu
    </button>
  )}
</ColorPicker>
```

---

### RULE-COLOR-PICKER-07: Sử dụng `width`/`height` để thay đổi kích thước nút — không override qua className

Nút color picker được render là `IconButton` với kích thước `width-8` x `height-8` (trừ padding). Không override kích thước qua className.

**Sai:**

```tsx
// ❌ Override kích thước qua className
<ColorPicker
  defaultValue={color}
  onChangeColor={setColor}
  className={cx("w-[24px] h-[24px]")}
/>
```

**Đúng:**

```tsx
// ✅ Nút nhỏ
<ColorPicker defaultValue={color} onChangeColor={setColor} width={24} height={24} />

// ✅ Nút lớn
<ColorPicker defaultValue={color} onChangeColor={setColor} width={60} height={60} />
```
