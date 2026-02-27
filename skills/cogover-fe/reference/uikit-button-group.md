---
title: Cách sử dụng ButtonGroup Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng ButtonGroup sai dẫn đến sizing không nhất quán, border styles bị hỏng, và phải tự propagate size một cách không cần thiết
tags: button-group, button, size, row, col, ui-kit
---

## Cách sử dụng ButtonGroup Component của ui-kit

Đường dẫn component: `ui-kit/src/components/ButtonGroup`

Props: `size`, `row`. Extends `React.HTMLAttributes<HTMLDivElement>` và hỗ trợ `ref` forwarding.

### RULE-BTN-GROUP-01: Sử dụng `ButtonGroup` để nhóm các button liên quan một cách trực quan

Khi render một tập các button liên quan cạnh nhau, sử dụng `ButtonGroup` thay vì flex wrapper thủ công. Nó tự động xử lý việc nối border, border-radius ở các cạnh, và propagate size.

**Sai:**
```tsx
// ❌ Manual grouping làm mất border-joining và size sync
<div className={cx("flex")}>
  <Button size="md">Left</Button>
  <Button size="md">Center</Button>
  <Button size="md">Right</Button>
</div>
```

**Đúng:**
```tsx
// ✅ ButtonGroup xử lý borders và size cho tất cả children
<ButtonGroup size="md">
  <Button>Left</Button>
  <Button>Center</Button>
  <Button>Right</Button>
</ButtonGroup>
```

### RULE-BTN-GROUP-02: Set `size` trên `ButtonGroup`, không phải trên từng children

`ButtonGroup` tự động inject prop `size` vào mọi child button. Việc set `size` trên từng button bên trong `ButtonGroup` là dư thừa và sẽ bị override.

**Sai:**
```tsx
// ❌ size trên children bị override bởi ButtonGroup anyway
<ButtonGroup>
  <Button size="sm">A</Button>
  <Button size="sm">B</Button>
</ButtonGroup>
```

**Đúng:**
```tsx
// ✅ Set size một lần trên group
<ButtonGroup size="sm">
  <Button>A</Button>
  <Button>B</Button>
</ButtonGroup>
```

### RULE-BTN-GROUP-03: Sử dụng `row={false}` cho vertical layout

Layout mặc định là horizontal (`row={true}`). Truyền `row={false}` để xếp buttons theo chiều dọc.

```tsx
// ✅ Vertical button group
<ButtonGroup size="md" row={false}>
  <Button>Top</Button>
  <Button>Middle</Button>
  <Button>Bottom</Button>
</ButtonGroup>
```

### RULE-BTN-GROUP-04: Không nest `ButtonGroup` bên trong `ButtonGroup` khác

Việc nest các group phá vỡ logic border và border-radius được xử lý bởi CSS selectors (`first:`, `last-of-type:`).

**Sai:**
```tsx
// ❌ Nested groups phá vỡ border styling
<ButtonGroup size="md">
  <ButtonGroup>
    <Button>A</Button>
  </ButtonGroup>
  <Button>B</Button>
</ButtonGroup>
```

**Đúng:**
```tsx
// ✅ Chỉ flat children
<ButtonGroup size="md">
  <Button>A</Button>
  <Button>B</Button>
</ButtonGroup>
```
