## Alert Component (`uikit-alert`)

Đường dẫn: `ui-kit/src/components/Alert`

Props: `title`, `message`, `icon`, `variant` ("warning"|"info"), `className`, `messageClassName`, `titleClassName`.

| Variant | Background | Border left | Dùng cho |
|---------|-----------|-------------|----------|
| `warning` (mặc định) | `bg-secondary-light-95` | `bg-secondary-main` | Cảnh báo, lưu ý quan trọng |
| `info` | `bg-blue-50` | `bg-info` | Thông tin bổ sung, hướng dẫn |

---

### RULE-ALERT-01: Chọn `variant` phù hợp với ngữ cảnh

```tsx
// ❌ Warning cho thông tin hướng dẫn thông thường
<Alert variant="warning" message="Vui lòng chọn template để xuất file" />

// ✅
<Alert variant="info" message="Vui lòng chọn template để xuất file" />
<Alert title="Thông báo" message="Số lượng bản ghi đã vượt quá giới hạn" />  // warning là mặc định
```

---

### RULE-ALERT-02: Không truyền rõ ràng giá trị mặc định

```tsx
// ❌ Thừa
<Alert variant="warning" icon={<WarningIcon />} message="Đây là cảnh báo" />

// ✅
<Alert message="Đây là cảnh báo" />
<Alert variant="info" message="Thông tin bổ sung" />
```

---

### RULE-ALERT-03: Truyền `icon={null}` để ẩn icon hoàn toàn

```tsx
// ❌ Fragment/string rỗng vẫn render wrapper icon
<Alert icon={<></>} message="Thông báo" />

// ✅
<Alert icon={null} variant="info" message="Vui lòng chọn template" />
```

---

### RULE-ALERT-04: Dùng `title` khi cần phân cấp tiêu đề/nội dung

```tsx
// ❌ Đặt tiêu đề vào message — mất phân cấp
<Alert message={<><strong>Thông báo</strong>: Số lượng bản ghi đã vượt quá giới hạn</>} />

// ✅
<Alert title="Thông báo" message="Số lượng bản ghi đã vượt quá giới hạn cho phép" />
```

---

### RULE-ALERT-05: Dùng `messageClassName`/`titleClassName` để tuỳ chỉnh style nội dung

```tsx
// ❌ Override qua className container — không đúng cách
<Alert className="[&>div>div:last-child]:text-red-500" message="Lỗi xảy ra" />

// ✅
<Alert messageClassName={cx("text-danger")} message="Lỗi xảy ra" />
<Alert titleClassName={cx("text-danger")} title="Lỗi" message="Không thể lưu dữ liệu" />
<Alert className={cx("mt-[1rem]")} message="Lưu ý quan trọng" />  // className cho container
```
