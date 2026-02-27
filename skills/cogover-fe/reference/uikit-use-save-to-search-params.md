---
title: Cách sử dụng hook useSaveToSearchParams trong ui-kit
impact: HIGH
impactDescription: Hook lưu state vào URL search params với debounce — tự cập nhật URL gây lag và mất dữ liệu search params hiện có
tags: hook, search-params, url, save, debounce, sync, ui-kit
---

## Cách sử dụng hook useSaveToSearchParams trong ui-kit

Đường dẫn hook: `ui-kit/src/hooks/useSaveToSearchParams`

Hook tự động lưu data vào URL search parameters với debounce 800ms. Merge với search params hiện có, loại bỏ giá trị null/undefined/rỗng.

### Props

| Prop | Kiểu | Mặc định | Mô tả |
| --- | --- | --- | --- |
| `data` | `Record<string, string \| number \| null \| undefined>` | — | Dữ liệu cần lưu vào URL (bắt buộc) |
| `disabled` | `boolean` | `false` | Tắt cập nhật URL |

### RULE-STSP-01: Truyền object data — hook tự merge với search params hiện có và loại bỏ giá trị rỗng

```tsx
// ✅ Tự động merge và clean
useSaveToSearchParams({
    data: { search: "hello", status: "active", page: null }, // page sẽ bị xóa khỏi URL
});
```

### RULE-STSP-02: Không tự cập nhật URL bằng `navigate` hoặc `history.push` — hook đã dùng `navigate({ replace: true })` với debounce

```tsx
// ❌ Tự cập nhật URL
useEffect(() => {
    const params = new URLSearchParams(data);
    navigate(`?${params.toString()}`, { replace: true });
}, [data]);

// ✅ Sử dụng hook — tự debounce 800ms và replace history
useSaveToSearchParams({ data: formValues });
```

### RULE-STSP-03: Sử dụng `disabled` để tạm dừng cập nhật URL — không remove hook khỏi render tree

```tsx
// ✅ Tạm dừng sync
useSaveToSearchParams({
    data: formValues,
    disabled: isModalOpen, // Không cập nhật URL khi modal mở
});
```

### RULE-STSP-04: Kết hợp với `useGetStateFromSearchParams` để tạo two-way binding URL ↔ State

```tsx
// Đọc state từ URL
const { state } = useGetStateFromSearchParams({
    getState: (params) => ({
        search: params.search ?? "",
        status: params.status ?? "all",
    }),
});

// Ghi form values vào URL
const formValues = useWatch({ control });
useSaveToSearchParams({ data: formValues });
```

### RULE-STSP-05: Hook bỏ qua lần render đầu tiên — không lo URL bị cập nhật khi component mount

Hook sử dụng `useIgnoreFirstRenderEffect` nên chỉ cập nhật URL khi data thực sự thay đổi sau lần render đầu.
