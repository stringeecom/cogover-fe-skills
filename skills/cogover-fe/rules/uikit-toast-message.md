---
title: Cách sử dụng component ToastMessage trong ui-kit
impact: HIGH
impactDescription: Setup sai khiến toasts không bao giờ xuất hiện, xuất hiện ở vị trí sai hoặc auto-close không hoạt động
tags: toast, notification, alert, ui-kit
---

## Cách sử dụng component ToastMessage trong ui-kit

Đường dẫn component: `ui-kit/src/components/ToastMessage`

Exports: `ToastContainer`, `showToastMessage`, `showBrowserNotification`.

---

### RULE-TOAST-01: Mount `ToastContainer` chính xác một lần tại app root — không bao giờ bên trong một page component

`ToastContainer` lắng nghe sự kiện DOM toàn cục `stringee-toast-message::show`. Mount nó bên trong một page component có nghĩa là nó unmount khi navigate đi, khiến toasts ngừng hoạt động.

**Sai:**

```tsx
// ❌ Bên trong một page — unmount khi navigation
function SettingsPage() {
  return (
    <>
      <ToastContainer />
      <SettingsForm />
    </>
  );
}
```

**Đúng:**

```tsx
// ✅ Tại app root — luôn được mount
function App() {
  return (
    <>
      <Router>...</Router>
      <ToastContainer placement="bottom-left" autoCloseInterval={8000} />
    </>
  );
}
```

---

### RULE-TOAST-02: Sử dụng `showToastMessage` để trigger toasts — không bao giờ render `ToastMessage` trực tiếp

`ToastMessage` là component UI toast riêng lẻ. Render nó trực tiếp bỏ qua queue, auto-close và hệ thống animation được quản lý bởi `ToastContainer`.

**Sai:**

```tsx
// ❌ Render ToastMessage trực tiếp — không có animation, không có auto-close, không có queue
{
  showMessage && (
    <ToastMessage
      title="Saved!"
      type="success"
      onClose={() => setShowMessage(false)}
    />
  );
}
```

**Đúng:**

```tsx
// ✅ Sử dụng showToastMessage — được xử lý bởi ToastContainer
import { showToastMessage } from "ui-kit";

showToastMessage({ title: "Saved successfully!", type: "success" });
showToastMessage({
  title: "An error occurred",
  message: "Please try again.",
  type: "error",
});
showToastMessage({ title: "Note", type: "info" });
showToastMessage({ title: "Check your input", type: "warning" });
```

---

### RULE-TOAST-03: Khớp `placement` trong `showToastMessage` với `ToastContainer` đã mount

`ToastContainer` chỉ hiển thị các toasts có `placement` khớp với prop `placement` của riêng nó. Nếu chúng không khớp, toast bị bỏ qua âm thầm.

**Sai:**

```tsx
// ❌ Container là bottom-left nhưng toast target bottom-right — toast không bao giờ xuất hiện
<ToastContainer placement="bottom-left" />;

showToastMessage({ title: "Hello", placement: "bottom-right" });
```

**Đúng:**

```tsx
// ✅ Placements khớp
<ToastContainer placement="bottom-right" />;

showToastMessage({ title: "Hello", placement: "bottom-right" });

// ✅ Placement mặc định là "bottom-left" cho cả hai — không cần truyền placement rõ ràng
<ToastContainer />;
showToastMessage({ title: "Hello" });
```

---

### RULE-TOAST-04: Sử dụng `type` để truyền đạt ý nghĩa ngữ nghĩa — không sử dụng `"default"` cho các tin nhắn phản hồi

`"default"` không render icon và không có màu trên title. Luôn sử dụng một type ngữ nghĩa cho phản hồi user-facing.

**Sai:**

```tsx
// ❌ default type cho một hành động thành công — không có phản hồi trực quan
showToastMessage({ title: "Record saved", type: "default" });
```

**Đúng:**

```tsx
// ✅ Sử dụng type ngữ nghĩa phù hợp
showToastMessage({ title: "Record saved", type: "success" });
showToastMessage({ title: "Validation failed", type: "error" });
showToastMessage({ title: "Session expiring soon", type: "warning" });
showToastMessage({ title: "Sync in progress", type: "info" });
```

---

### RULE-TOAST-05: `showToastMessage` có thể được gọi bên ngoài React — không cần context hoặc hook

Function dispatch một custom event DOM. Nó hoạt động trong code không phải React (axios interceptors, service layers, utility functions) mà không cần bất kỳ React context nào.

**Đúng:**

```ts
// ✅ Trong một axios interceptor
axiosInstance.interceptors.response.use(
  (response) => response,
  (error) => {
    showToastMessage({ title: "Request failed", type: "error" });
    return Promise.reject(error);
  },
);
```

---

### RULE-TOAST-06: Sử dụng `showBrowserNotification` cho native OS notifications — không thay thế cho toasts

`showBrowserNotification` trigger browser's native Notification API. Nó yêu cầu user permission và dự định cho các alerts background/out-of-focus. Không sử dụng nó như một thay thế cho phản hồi toast in-app.

**Đúng:**

```tsx
// ✅ Sử dụng cho các alerts out-of-focus (ví dụ: cuộc gọi đến, tin nhắn mới khi tab ở background)
showBrowserNotification({
  title: "New message from John",
  message: "Hey, are you available?",
  targetUrl: "/messages/123",
  action: "focus",
});

// ✅ Sử dụng showToastMessage cho phản hồi in-app
showToastMessage({ title: "Message sent", type: "success" });
```
