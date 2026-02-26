---
title: Cách sử dụng hook useGetStateFromSearchParams trong ui-kit
impact: HIGH
impactDescription: Hook chuyển đổi URL search params thành state — tự parse sẽ mất type safety và không reactive khi URL thay đổi
tags: hook, search-params, url, state, query-string, ui-kit
---

## Cách sử dụng hook useGetStateFromSearchParams trong ui-kit

Đường dẫn hook: `ui-kit/src/hooks/useGetStateFromSearchParams`

Hook trích xuất và quản lý state từ URL search parameters với hàm chuyển đổi tùy chỉnh.

### Props

| Prop | Kiểu | Mô tả |
| --- | --- | --- |
| `getState` | `(searchParams: Record<string, string>) => T` | Hàm chuyển đổi search params thành state (bắt buộc) |

### Giá trị trả về

| Giá trị | Kiểu | Mô tả |
| --- | --- | --- |
| `state` | `T` | State đã chuyển đổi từ search params |

### RULE-GSSP-01: Truyền hàm `getState` để chuyển đổi search params thành typed state

```tsx
interface FilterState {
    search: string;
    status: string;
    page: number;
}

const { state } = useGetStateFromSearchParams<FilterState>({
    getState: (params) => ({
        search: params.search ?? "",
        status: params.status ?? "all",
        page: Number(params.page) || 1,
    }),
});
```

### RULE-GSSP-02: Không tự parse `location.search` thủ công — hook đã dùng `qs.parse` và reactive với URL

```tsx
// ❌ Tự parse
const params = new URLSearchParams(location.search);
const search = params.get("search") ?? "";

// ✅ Sử dụng hook — tự động cập nhật khi URL thay đổi
const { state } = useGetStateFromSearchParams({
    getState: (params) => ({ search: params.search ?? "" }),
});
```

### RULE-GSSP-03: Kết hợp với `useSaveToSearchParams` để tạo two-way binding giữa state và URL

```tsx
// Đọc state từ URL
const { state: filterState } = useGetStateFromSearchParams({
    getState: (params) => ({
        search: params.search ?? "",
        status: params.status ?? "all",
    }),
});

// Ghi state vào URL
useSaveToSearchParams({ data: formValues });
```

### RULE-GSSP-04: Hàm `getState` nên cung cấp giá trị mặc định cho mỗi field — search params có thể rỗng

```tsx
// ✅ Luôn có default values
const { state } = useGetStateFromSearchParams({
    getState: (params) => ({
        search: params.search ?? "",
        page: Number(params.page) || 1,
        perPage: Number(params.perPage) || 20,
    }),
});
```
