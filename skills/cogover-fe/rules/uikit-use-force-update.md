---
title: Cách sử dụng hook useForceUpdate trong ui-kit
impact: LOW
impactDescription: Force re-render component khi React dependency tracking không capture được thay đổi cần thiết
tags: hook, force-update, re-render, state, ui-kit
---

## Cách sử dụng hook useForceUpdate trong ui-kit

Đường dẫn hook: `ui-kit/src/hooks/useForceUpdate`

Hook cung cấp cơ chế buộc React component re-render bằng cách tăng bộ đếm state nội bộ.

### Giá trị trả về

| Giá trị | Kiểu | Mô tả |
| --- | --- | --- |
| `forceUpdate` | `() => void` | Hàm trigger re-render |
| `forceUpdateState` | `number` | Giá trị bộ đếm hiện tại (tăng mỗi lần gọi) |

### RULE-FU-01: Sử dụng `useForceUpdate` thay vì tự tạo state counter

```tsx
// ❌ Tự tạo logic force update
const [, setTick] = useState(0);
const forceUpdate = () => setTick(prev => prev + 1);

// ✅ Sử dụng hook
const { forceUpdate } = useForceUpdate();
```

### RULE-FU-02: Sử dụng `forceUpdateState` làm dependency khi cần trigger effect sau re-render

```tsx
// ✅ Dùng forceUpdateState trong dependency array
const { forceUpdate, forceUpdateState } = useForceUpdate();

useEffect(() => {
    // Logic cần chạy lại sau force update
}, [forceUpdateState]);
```

### RULE-FU-03: Chỉ sử dụng khi React dependency tracking không đủ — ưu tiên quản lý state đúng cách trước

Đây là hook escape hatch — ưu tiên cập nhật state/props đúng cách thay vì force re-render.
