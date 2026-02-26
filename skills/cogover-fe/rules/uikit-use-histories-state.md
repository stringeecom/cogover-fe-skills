---
title: Cách sử dụng hook useHistoriesState trong ui-kit
impact: HIGH
impactDescription: Hook quản lý undo/redo — tự implement sẽ thiếu keyboard shortcuts, duplicate detection, và edge case handling
tags: hook, undo, redo, history, state, keyboard-shortcuts, ui-kit
---

## Cách sử dụng hook useHistoriesState trong ui-kit

Đường dẫn hook: `ui-kit/src/hooks/useHistoriesState`

Hook quản lý hệ thống lịch sử state (undo/redo) với hỗ trợ keyboard shortcuts.

### Props

| Prop | Kiểu | Mặc định | Mô tả |
| --- | --- | --- | --- |
| `initState` | `T` | — | Giá trị state khởi tạo (bắt buộc) |
| `enableShortcuts` | `boolean` | `false` | Bật keyboard shortcuts (Ctrl/Cmd+Z, Ctrl/Cmd+Shift+Z) |

### Giá trị trả về

| Giá trị | Kiểu | Mô tả |
| --- | --- | --- |
| `state` | `T` | State hiện tại |
| `back` | `() => void` | Undo |
| `canBack` | `boolean` | Có thể undo không |
| `forward` | `() => void` | Redo |
| `canForward` | `boolean` | Có thể redo không |
| `setIndicate` | `(index: number) => void` | Nhảy đến vị trí lịch sử cụ thể |
| `replace` | `(newState: T) => void` | Thay thế entry hiện tại (không tạo lịch sử mới) |
| `push` | `(newState: T) => void` | Thêm entry mới (xóa forward history) |
| `shortcutsEnabled` | `boolean` | Trạng thái shortcuts |
| `toggleShortcuts` | `(enabled?: boolean) => void` | Bật/tắt shortcuts |

### RULE-HS-01: Sử dụng `push` để thêm state mới và `replace` để cập nhật state hiện tại — không nhầm lẫn

```tsx
const { push, replace } = useHistoriesState({ initState: initialData });

// ✅ push: tạo entry mới trong lịch sử (cho phép undo về trước)
push(newData);

// ✅ replace: cập nhật entry hiện tại (không tạo lịch sử)
replace(updatedData);
```

### RULE-HS-02: Kiểm tra `canBack`/`canForward` trước khi hiển thị nút undo/redo

```tsx
const { back, canBack, forward, canForward } = useHistoriesState({ initState });

<Button onClick={back} disabled={!canBack}>Undo</Button>
<Button onClick={forward} disabled={!canForward}>Redo</Button>
```

### RULE-HS-03: Bật `enableShortcuts` để tự động hỗ trợ Ctrl/Cmd+Z (undo) và Ctrl/Cmd+Shift+Z (redo)

```tsx
// ✅ Tự động đăng ký keyboard shortcuts
const history = useHistoriesState({
    initState: initialData,
    enableShortcuts: true,
});
```

### RULE-HS-04: Sử dụng `toggleShortcuts` để tạm tắt shortcuts khi component khác cần xử lý phím tắt

```tsx
const { toggleShortcuts } = useHistoriesState({ initState, enableShortcuts: true });

// Tắt tạm khi modal mở
const handleModalOpen = () => toggleShortcuts(false);
const handleModalClose = () => toggleShortcuts(true);
```

### RULE-HS-05: Hook tự động loại bỏ state trùng lặp bằng deep equal — không cần tự kiểm tra trước khi push

```tsx
// ✅ Hook dùng lodash.isEqual để so sánh — push giá trị trùng sẽ bị bỏ qua
push(sameData); // Không tạo entry mới nếu trùng state hiện tại
```

### RULE-HS-06: Không tự implement undo/redo với mảng state — sử dụng hook này

```tsx
// ❌ Tự implement
const [history, setHistory] = useState([initState]);
const [index, setIndex] = useState(0);
const undo = () => setIndex(i => Math.max(0, i - 1));

// ✅ Sử dụng hook
const { state, back, forward, push } = useHistoriesState({ initState });
```
