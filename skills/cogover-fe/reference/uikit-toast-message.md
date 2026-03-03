## Toast Message System (`uikit-toast-message`)

Đường dẫn: `ui-kit/src/components/ToastMessage`

Hệ thống toast gồm 3 phần: **ToastContainer** (container quản lý hiển thị), **ToastMessage** (UI đơn lẻ), **showToastMessage** (hàm trigger).

---

### Kiến trúc

```
showToastMessage(options)
  → dispatch CustomEvent "stringee-toast-message::show"
    → ToastContainer lắng nghe → render ToastMessage(s)
```

- `ToastContainer` phải mount ở root layout (mỗi `placement` cần 1 instance).
- `showToastMessage` gọi ở bất kỳ đâu — không cần import container.

---

### showToastMessage

```tsx
import { showToastMessage } from "@stringeecom/ui-kit";

showToastMessage({
  type: "success",           // "default" | "success" | "warning" | "error" | "info"
  title: "Tạo thành công",  // ReactNode (bắt buộc)
  message: "Bản ghi đã được thêm vào hệ thống", // ReactNode (tuỳ chọn)
  size: "medium",            // "small" | "medium" | "large" (mặc định "medium")
  placement: "bottom-left",  // mặc định "bottom-left"
  autoCloseInterval: 5000,   // ms, ghi đè giá trị container (tuỳ chọn)
});
```

| Param | Type | Default | Mô tả |
|-------|------|---------|-------|
| `title` | `ReactNode` | — | Tiêu đề toast (bắt buộc) |
| `message` | `ReactNode` | — | Nội dung chi tiết |
| `type` | `"default" \| "success" \| "warning" \| "error" \| "info"` | `"default"` | Loại toast (icon + màu) |
| `size` | `"small" \| "medium" \| "large"` | `"medium"` | Kích thước toast |
| `placement` | `ToastMessagePlacement` | `"bottom-left"` | Vị trí hiển thị |
| `autoCloseInterval` | `number` | — | Thời gian tự đóng (ms), ghi đè container |

**Size mapping:**

| Size | Width | Padding | Message max-width |
|------|-------|---------|-------------------|
| `small` | 280px | `px-[0.5rem] py-[0.75rem]` | 220px |
| `medium` | 320px | `px-[0.75rem] py-[1.25rem]` | 260px |
| `large` | 400px | `px-[1rem] py-[1.5rem]` | 340px |

---

### ToastContainer

```tsx
import { ToastContainer } from "@stringeecom/ui-kit";

// Mount ở root layout — mỗi placement cần 1 instance
<ToastContainer placement="bottom-left" />
<ToastContainer placement="top-right" autoCloseInterval={5000} />
```

| Prop | Type | Default | Mô tả |
|------|------|---------|-------|
| `placement` | `ToastMessagePlacement` | `"bottom-left"` | Vị trí container |
| `autoCloseInterval` | `number` | `8000` | Thời gian auto-close mặc định (ms) |
| `className` | `string` | — | Class bổ sung cho toast items |
| `closeAll` | `boolean` | `true` | Hiện nút "Đóng tất cả" khi > 1 toast |

**Placement options:** `"top-left"` · `"top-center"` · `"top-right"` · `"bottom-left"` · `"bottom-center"` · `"bottom-right"`

---

### ToastMessage (component đơn lẻ)

Thường không dùng trực tiếp — ToastContainer tự render. Chỉ dùng khi cần toast tĩnh trong layout.

```tsx
import { ToastMessage } from "@stringeecom/ui-kit";

<ToastMessage
  title="Thông báo"
  message="Nội dung chi tiết"
  type="info"
  onClose={() => {}}
/>
```

| Prop | Type | Default | Mô tả |
|------|------|---------|-------|
| `title` | `ReactNode` | — | Tiêu đề (bắt buộc) |
| `message` | `ReactNode` | — | Nội dung chi tiết |
| `type` | `ToastMessageTypes` | `"default"` | Loại toast |
| `size` | `ToastMessageSize` | `"medium"` | Kích thước toast |
| `onClose` | `() => void` | — | Callback đóng (hiện nút X khi truyền) |
| `onClick` | `(e) => void` | — | Click handler |
| `...divProps` | `HTMLAttributes` | — | Mọi div attribute khác |

---

### showBrowserNotification

```tsx
import { showBrowserNotification } from "@stringeecom/ui-kit";

showBrowserNotification({
  title: "Tin nhắn mới",
  message: "Bạn có 1 tin nhắn từ khách hàng",
  targetUrl: "/messages/123",
  action: "focus",  // "focus" | "open_new_tab"
});
```

| Param | Type | Default | Mô tả |
|-------|------|---------|-------|
| `title` | `string` | — | Tiêu đề notification (bắt buộc) |
| `message` | `string` | — | Nội dung |
| `icon` | `string` | Logo Stringee | URL icon |
| `targetUrl` | `string` | — | URL chuyển hướng khi click |
| `action` | `"focus" \| "open_new_tab"` | `"focus"` | Hành vi khi click |

---

### RULE-TOAST-01: Dùng `showToastMessage`, không render ToastMessage trực tiếp

```tsx
// ❌ Tự render toast
const [showToast, setShowToast] = useState(false);
{showToast && <ToastMessage title="Thành công" type="success" />}

// ✅ Gọi hàm — container xử lý hiển thị + animation
showToastMessage({ type: "success", title: "Tạo thành công" });
```

---

### RULE-TOAST-02: Chọn `type` phù hợp ngữ cảnh

```tsx
// ❌ Dùng "default" cho mọi trường hợp
showToastMessage({ type: "default", title: "Đã xoá bản ghi" });

// ✅ Dùng type phản ánh kết quả
showToastMessage({ type: "success", title: "Tạo thành công", message: "Bản ghi đã được thêm" });
showToastMessage({ type: "error", title: "Lỗi", message: "Không thể xoá bản ghi" });
showToastMessage({ type: "warning", title: "Cảnh báo", message: "Dữ liệu có thể bị mất" });
showToastMessage({ type: "info", title: "Thông tin", message: "Hệ thống sẽ bảo trì lúc 22:00" });
```

---

### RULE-TOAST-03: Luôn truyền `title`, `message` là tuỳ chọn

```tsx
// ❌ Chỉ có message, không có title
showToastMessage({ type: "success", message: "Đã lưu" });

// ✅ Title bắt buộc
showToastMessage({ type: "success", title: "Đã lưu" });
showToastMessage({ type: "success", title: "Đã lưu", message: "Thay đổi đã được áp dụng" });
```

---

### RULE-TOAST-04: Dùng `autoCloseInterval` per-message khi cần thời gian khác container

```tsx
// ❌ Đổi container interval cho 1 toast đặc biệt
<ToastContainer autoCloseInterval={3000} />

// ✅ Ghi đè per-message
showToastMessage({
  type: "error",
  title: "Lỗi nghiêm trọng",
  message: "Chi tiết lỗi dài...",
  autoCloseInterval: 15000, // hiển thị lâu hơn
});
```

---

### RULE-TOAST-05: Dùng class `hide-on-click` cho phần tử cần đóng toast khi click

```tsx
// ✅ Nút trong toast content — click sẽ tự đóng toast
showToastMessage({
  type: "success",
  title: "Đã tạo bản ghi",
  message: (
    <div>
      <a href="/records/123" className="hide-on-click">Xem chi tiết</a>
    </div>
  ),
});
```

---

### RULE-TOAST-06: Không mount nhiều ToastContainer cùng placement

```tsx
// ❌ Duplicate container → toast hiện 2 lần
<ToastContainer placement="bottom-left" />
<ToastContainer placement="bottom-left" />

// ✅ Mỗi placement chỉ 1 instance
<ToastContainer placement="bottom-left" />
<ToastContainer placement="top-right" />
```

### RULE-TOAST-07: Chọn `size` phù hợp với nội dung

```tsx
// ❌ Dùng "large" cho toast chỉ có title ngắn
showToastMessage({ type: "success", title: "Đã lưu", size: "large" });

// ❌ Truyền size="medium" (đã là mặc định)
showToastMessage({ type: "success", title: "Đã lưu", size: "medium" });

// ✅ Không truyền size khi dùng mặc định
showToastMessage({ type: "success", title: "Đã lưu" });

// ✅ "small" cho toast ngắn gọn, chỉ title
showToastMessage({ type: "success", title: "Đã lưu", size: "small" });

// ✅ "large" cho toast có nội dung dài hoặc chứa action
showToastMessage({
  type: "error",
  title: "Lỗi import dữ liệu",
  size: "large",
  message: (
    <div>
      <p>3 bản ghi không hợp lệ. Vui lòng kiểm tra lại file và thử lại.</p>
      <a href="/import/logs" className="hide-on-click">Xem chi tiết</a>
    </div>
  ),
});
```

---

**DO:** Import `showToastMessage` từ `@stringeecom/ui-kit`. Dùng `type` phù hợp ngữ cảnh. Truyền `title` rõ ràng. Mount `ToastContainer` ở root layout. Chọn `size` phù hợp nội dung.

**DON'T:** Render `ToastMessage` trực tiếp thay vì gọi `showToastMessage`. Mount nhiều container cùng `placement`. Dùng `type="default"` khi có type cụ thể hơn. Bỏ qua `title`. Truyền `size="medium"` (đã là mặc định).
