---
title: Cách sử dụng Alert Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng Alert sai dẫn đến việc hiển thị icon dư thừa, variant không phù hợp và custom class không đúng chỗ
tags: alert, warning, info, notification, ui-kit
---

## Cách sử dụng Alert Component của ui-kit

Đường dẫn component: `ui-kit/src/components/Alert`

Props: `title`, `message`, `icon`, `variant`, `className`, `messageClassName`, `titleClassName`.

---

### RULE-ALERT-01: Sử dụng `variant` phù hợp với ngữ cảnh thông báo

Alert hỗ trợ 2 variant: `"warning"` (mặc định) và `"info"`. Sử dụng `"warning"` cho các cảnh báo, lưu ý quan trọng. Sử dụng `"info"` cho các thông tin bổ sung, hướng dẫn.

**Sai:**

```tsx
// ❌ Dùng variant warning cho thông tin hướng dẫn thông thường
<Alert
  variant="warning"
  message="Vui lòng chọn template để xuất file"
/>
```

**Đúng:**

```tsx
// ✅ Dùng variant info cho thông tin hướng dẫn
<Alert
  variant="info"
  message="Vui lòng chọn template để xuất file"
/>

// ✅ Dùng variant warning (hoặc bỏ qua vì là mặc định) cho cảnh báo
<Alert
  title="Thông báo"
  message="Số lượng bản ghi đã vượt quá giới hạn cho phép"
/>
```

---

### RULE-ALERT-02: Không truyền rõ ràng giá trị mặc định cho `variant` và `icon`

`variant` mặc định là `"warning"` và `icon` mặc định là icon cảnh báo (tam giác vàng). Chỉ truyền khi cần thay đổi giá trị mặc định.

**Sai:**

```tsx
// ❌ Thừa — variant và icon đã có giá trị mặc định
<Alert
  variant="warning"
  icon={<WarningIcon />}
  message="Đây là cảnh báo"
/>
```

**Đúng:**

```tsx
// ✅ Bỏ qua khi dùng giá trị mặc định
<Alert message="Đây là cảnh báo" />

// ✅ Chỉ truyền khi cần thay đổi
<Alert variant="info" message="Thông tin bổ sung" />
```

---

### RULE-ALERT-03: Truyền `icon={null}` để ẩn icon, không truyền icon rỗng

Khi không muốn hiển thị icon, truyền `icon={null}`. Không truyền chuỗi rỗng hoặc fragment rỗng.

**Sai:**

```tsx
// ❌ Fragment rỗng vẫn render wrapper của icon
<Alert icon={<></>} message="Thông báo" />

// ❌ Chuỗi rỗng vẫn render wrapper của icon
<Alert icon="" message="Thông báo" />
```

**Đúng:**

```tsx
// ✅ null sẽ ẩn hoàn toàn phần icon
<Alert icon={null} variant="info" message="Vui lòng chọn template để xuất file" />
```

---

### RULE-ALERT-04: Sử dụng `title` khi cần phân biệt tiêu đề và nội dung chi tiết

`title` là optional. Chỉ sử dụng khi cần hiển thị tiêu đề in đậm phía trên nội dung message. Khi có `title`, layout sẽ không căn giữa theo chiều dọc nữa và message sẽ có khoảng cách phía trên.

**Sai:**

```tsx
// ❌ Đặt tiêu đề vào message — mất phân cấp thông tin
<Alert message={<><strong>Thông báo</strong>: Số lượng bản ghi đã vượt quá giới hạn</>} />
```

**Đúng:**

```tsx
// ✅ Sử dụng title để phân cấp thông tin rõ ràng
<Alert
  title="Thông báo"
  message="Số lượng bản ghi đã vượt quá giới hạn cho phép"
/>

// ✅ Chỉ có message khi không cần tiêu đề
<Alert message="Lưu ý: Hành động này không thể hoàn tác" />
```

---

### RULE-ALERT-05: Sử dụng `messageClassName` và `titleClassName` để tuỳ chỉnh style, không override bằng `className`

`className` áp dụng cho container ngoài cùng. Khi cần tuỳ chỉnh style của nội dung message hoặc title, sử dụng `messageClassName` và `titleClassName` tương ứng.

**Sai:**

```tsx
// ❌ Override style message qua className của container
<Alert
  className="[&>div>div:last-child]:text-red-500"
  message="Lỗi xảy ra"
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng messageClassName để tuỳ chỉnh style message
<Alert
  messageClassName={cx("text-danger")}
  message="Lỗi xảy ra"
/>

// ✅ Sử dụng titleClassName để tuỳ chỉnh style title
<Alert
  titleClassName={cx("text-danger")}
  title="Lỗi"
  message="Không thể lưu dữ liệu"
/>

// ✅ Sử dụng className cho container (ví dụ: thêm margin)
<Alert
  className={cx("mt-[1rem]")}
  message="Lưu ý quan trọng"
/>
```

---

## Tham chiếu style

| Variant | Background | Border left |
|---------|-----------|-------------|
| `warning` | `bg-secondary-light-95` | `bg-secondary-main` |
| `info` | `bg-blue-50` | `bg-info` |

| Phần tử | Typography class |
|---------|-----------------|
| Title | `prose-body1 text-typo-primary` |
| Message | `prose-caption2 text-typo-secondary` |
