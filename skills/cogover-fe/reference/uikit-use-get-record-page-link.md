---
title: Cách sử dụng hook useGetRecordPageLink trong ui-kit
impact: HIGH
impactDescription: Hook tạo link điều hướng record — hardcode route sẽ sai khi chuyển đổi giữa serial page và normal page
tags: hook, record, link, navigation, route, serial-page, ui-kit
---

## Cách sử dụng hook useGetRecordPageLink trong ui-kit

Đường dẫn hook: `ui-kit/src/hooks/useGetRecordPageLink`

Hook tạo link điều hướng cho các trang record, tự động điều chỉnh route dựa trên context (serial page vs normal page).

### Giá trị trả về

| Giá trị | Kiểu | Mô tả |
| --- | --- | --- |
| `listPageLink` | `(params: { objectSlug: string }) => string` | Tạo link trang danh sách |
| `createPageLink` | `(params: { objectSlug: string }) => string` | Tạo link trang tạo mới |
| `viewPageLink` | `(params: { objectSlug: string; recordId: string }) => string` | Tạo link trang xem chi tiết |

### RULE-GRPL-01: Luôn sử dụng `useGetRecordPageLink` thay vì hardcode route — hook tự phát hiện serial page context

```tsx
// ❌ Hardcode route — sai khi ở serial page
const link = `/record/${objectSlug}`;

// ✅ Hook tự điều chỉnh route
const { listPageLink } = useGetRecordPageLink();
const link = listPageLink({ objectSlug: "contacts" });
```

### RULE-GRPL-02: Sử dụng đúng hàm cho đúng mục đích — `listPageLink`, `createPageLink`, `viewPageLink`

```tsx
const { listPageLink, createPageLink, viewPageLink } = useGetRecordPageLink();

// Danh sách
const listLink = listPageLink({ objectSlug: "contacts" });

// Tạo mới
const createLink = createPageLink({ objectSlug: "contacts" });

// Xem chi tiết
const viewLink = viewPageLink({ objectSlug: "contacts", recordId: "123" });
```

### RULE-GRPL-03: Không tự kiểm tra pathname để xác định serial page — hook đã tự phát hiện `/s/` hoặc `/c/s/`

```tsx
// ❌ Tự kiểm tra
const isSerial = location.pathname.includes("/s/");
const link = isSerial ? `/s/record/${objectSlug}` : `/record/${objectSlug}`;

// ✅ Hook đã xử lý
const { listPageLink } = useGetRecordPageLink();
const link = listPageLink({ objectSlug });
```

### RULE-GRPL-04: Không tự sử dụng `generatePath` và `routeMapFullPath` — hook đã wrap sẵn

```tsx
// ❌ Tự sử dụng
import { generatePath, routeMapFullPath } from "utils/routeMap";
const link = generatePath(routeMapFullPath.record.list.index, { objectSlug });

// ✅ Sử dụng hook
const { listPageLink } = useGetRecordPageLink();
const link = listPageLink({ objectSlug });
```
