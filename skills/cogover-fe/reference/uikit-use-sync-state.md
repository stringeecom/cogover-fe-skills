---
title: Cách sử dụng hook useSyncState trong ui-kit
impact: MEDIUM
impactDescription: Hook đồng bộ state giữa các tab trình duyệt — tự implement BroadcastChannel sẽ thiếu cleanup và state sync
tags: hook, sync, state, broadcast-channel, tabs, cross-tab, ui-kit
---

## Cách sử dụng hook useSyncState trong ui-kit

Đường dẫn hook: `ui-kit/src/hooks/useSyncState`

Hook đồng bộ state giữa nhiều tab/window trình duyệt sử dụng BroadcastChannel API.

### Tham số

| Tham số | Kiểu | Mô tả |
| --- | --- | --- |
| `key` | `string` | Key duy nhất cho channel (bắt buộc) |
| `value` | `T` | Giá trị state khởi tạo (bắt buộc) |

### Giá trị trả về

| Giá trị | Kiểu | Mô tả |
| --- | --- | --- |
| `[0]` | `T` | State hiện tại |
| `[1]` | `(newValue: T) => void` | Hàm cập nhật state (đồng bộ sang các tab khác) |

### RULE-SS-01: API giống `useState` — destructure thành `[state, setState]`

```tsx
const [sharedState, setSharedState] = useSyncState("my-key", initialValue);

// Cập nhật — tự động sync sang các tab khác
setSharedState(newValue);
```

### RULE-SS-02: `key` phải duy nhất cho mỗi state cần đồng bộ — channel name là `sync-state-{key}`

```tsx
// ✅ Keys khác nhau cho states khác nhau
const [theme, setTheme] = useSyncState("theme", "light");
const [locale, setLocale] = useSyncState("locale", "vi");
```

### RULE-SS-03: Không tự implement BroadcastChannel — hook đã xử lý cleanup listener và close channel

```tsx
// ❌ Tự implement — thiếu cleanup
const bc = new BroadcastChannel("my-channel");
bc.onmessage = (e) => setState(e.data);

// ✅ Hook xử lý sẵn lifecycle
const [state, setState] = useSyncState("my-key", initialValue);
```

### RULE-SS-04: Khi `value` prop thay đổi, state local tự động cập nhật — không cần useEffect bổ sung

```tsx
// ✅ State tự sync khi value prop thay đổi
const [state, setState] = useSyncState("key", propValue);
// Khi propValue thay đổi → state tự cập nhật
```

### RULE-SS-05: Chỉ hoạt động giữa các tab cùng origin — không dùng cho cross-domain communication
