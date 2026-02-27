---
title: Quy tắc tạo hook TanStack Query
impact: HIGH
impactDescription: Tạo hook không đúng pattern dẫn đến code không nhất quán, khó bảo trì và không tương thích với codebase hiện tại
tags: tanstack-query, react-query, hook, useQuery, useMutation, api
---

## Quy tắc tạo hook TanStack Query

### RULE-TQ-01: Khi tạo các hook của TanStack Query, phải tham khảo và làm giống các hook trong thư mục `src/apis` của dự án hiện tại

Trước khi tạo bất kỳ hook nào sử dụng `useQuery`, `useMutation`, hoặc `useInfiniteQuery` từ `@tanstack/react-query`, phải đọc và tham khảo các hook đã có trong thư mục `src/apis` của dự án hiện tại để đảm bảo:

- **Cấu trúc file**: Đặt file đúng vị trí, đúng naming convention giống các file trong `src/apis`
- **Naming convention**: Đặt tên hook giống pattern đã có (ví dụ: `useQuery*` cho query, `use<Action><Entity>` cho mutation)
- **Query key**: Sử dụng cách quản lý query key giống các hook hiện tại
- **Type definition**: Định nghĩa type cho request/response giống các file type đã có
- **Error handling**: Xử lý lỗi giống pattern đã có
- **Options pattern**: Truyền options (onSuccess, onError, enabled...) giống cách các hook hiện tại đang làm

**Sai:**

```typescript
// ❌ Không tham khảo code hiện tại, tự viết theo cách riêng
const useGetUsers = () => {
  return useQuery({
    queryKey: ["users"],
    queryFn: () => fetch("/api/users").then((res) => res.json()),
  });
};
```

**Đúng:**

```typescript
// ✅ Đọc các hook trong src/apis trước, sau đó làm theo đúng pattern
// Ví dụ: nếu các hook hiện tại sử dụng axios instance, query key factory, và type riêng
// thì hook mới cũng phải làm tương tự
```
