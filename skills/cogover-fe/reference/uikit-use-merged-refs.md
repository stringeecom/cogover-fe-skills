---
title: Cách sử dụng hook useMergedRefs trong ui-kit
impact: MEDIUM
impactDescription: Hook merge nhiều refs vào một — tự xử lý ref forwarding sẽ bỏ sót function refs hoặc object refs
tags: hook, ref, merge, forward, callback-ref, ui-kit
---

## Cách sử dụng hook useMergeRefs trong ui-kit

Đường dẫn hook: `ui-kit/src/hooks/useMergeRefs`

Hook merge nhiều React refs (function refs và object refs) thành một ref callback duy nhất.

### Tham số

| Tham số | Kiểu | Mô tả |
| --- | --- | --- |
| `...refs` | `React.Ref<T>[]` | Danh sách refs cần merge (variadic) |

### Giá trị trả về

| Giá trị | Kiểu | Mô tả |
| --- | --- | --- |
| (trả về trực tiếp) | `(instance: T) => void` | Ref callback đã merge |

### RULE-MREFS-01: Sử dụng `useMergedRefs` khi cần gán nhiều refs cho cùng một element

```tsx
// ❌ Tự xử lý — dễ bỏ sót ref types
const ref1 = useRef<HTMLDivElement>(null);
<div ref={(el) => { ref1.current = el; forwardedRef(el); }} />

// ✅ Sử dụng hook
const ref1 = useRef<HTMLDivElement>(null);
const mergedRef = useMergedRefs(ref1, forwardedRef);
<div ref={mergedRef} />
```

### RULE-MREFS-02: Hook hỗ trợ cả function refs và object refs — không cần kiểm tra kiểu

```tsx
const objectRef = useRef<HTMLDivElement>(null);
const functionRef = (el: HTMLDivElement) => { /* custom logic */ };

// ✅ Trộn cả 2 loại
const mergedRef = useMergedRefs(objectRef, functionRef, forwardedRef);
```

### RULE-MREFS-03: Sử dụng phổ biến trong `forwardRef` components khi cần internal ref lẫn forwarded ref

```tsx
const MyComponent = forwardRef<HTMLDivElement, Props>((props, ref) => {
    const internalRef = useRef<HTMLDivElement>(null);
    const mergedRef = useMergedRefs(internalRef, ref);

    return <div ref={mergedRef} />;
});
```

### RULE-MREFS-04: Lưu ý tên export là `useMergedRefs` (có "d") — không phải `useMergeRefs`

```tsx
// ✅ Import đúng tên
import { useMergedRefs } from "@stringeecom/ui-kit";
```
