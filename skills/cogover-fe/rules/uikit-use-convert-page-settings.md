---
title: Cách sử dụng hook useConvertPageSettings trong ui-kit
impact: MEDIUM
impactDescription: Hook chuyển đổi page settings thành CSS styles — tự xử lý sẽ gây sai lệch layout và breadcrumb
tags: hook, page-settings, css, layout, breadcrumb, ui-kit
---

## Cách sử dụng hook useConvertPageSettings trong ui-kit

Đường dẫn hook: `ui-kit/src/hooks/useConvertPageSettings`

### Props

| Prop | Kiểu | Mặc định | Mô tả |
| --- | --- | --- | --- |
| `pageSettings` | `PageSettings \| string` | — | Settings object hoặc JSON string từ server (bắt buộc) |
| `fromModal` | `boolean` | `false` | Có đang render trong modal không |
| `editMode` | `boolean` | `false` | Chế độ chỉnh sửa (edit) hay tạo mới (create) |
| `isStandardLayout` | `boolean` | `false` | Có sử dụng standard layout không |

### Giá trị trả về

| Giá trị | Kiểu | Mô tả |
| --- | --- | --- |
| `settingPageStyles` | `Record<string, string>` | CSS styles cho container ngoài |
| `settingContentPageStyles` | `Record<string, string>` | CSS styles cho content bên trong |
| `settingBreadcrumbsPageStyles` | `object` | Cấu hình breadcrumb (isShow, isShowTitle, typeShowTitle, title) |

### RULE-CPS-01: Luôn truyền `pageSettings` — đây là prop bắt buộc duy nhất

```tsx
// ✅ Truyền object
const styles = useConvertPageSettings({ pageSettings: layoutResponse.pageSettings });

// ✅ Truyền JSON string
const styles = useConvertPageSettings({ pageSettings: jsonString });
```

### RULE-CPS-02: Sử dụng `fromModal={true}` khi render trong modal — hook sẽ trả về styles rỗng và breadcrumb đơn giản

```tsx
// ❌ Tự xử lý logic modal
const styles = useConvertPageSettings({ pageSettings });
const finalStyles = isModal ? {} : styles.settingPageStyles;

// ✅ Truyền fromModal để hook tự xử lý
const styles = useConvertPageSettings({ pageSettings, fromModal: true });
```

### RULE-CPS-03: Sử dụng `editMode` để phân biệt breadcrumb giữa create và edit mode

```tsx
// ✅ Create mode
const styles = useConvertPageSettings({ pageSettings, editMode: false });

// ✅ Edit mode
const styles = useConvertPageSettings({ pageSettings, editMode: true });
```

### RULE-CPS-04: Không tự parse JSON pageSettings hoặc tính toán CSS từ page settings — hook đã xử lý sẵn

```tsx
// ❌ Tự parse và tính toán
const parsed = JSON.parse(pageSettings);
const padding = `${parsed.padding.top}px ${parsed.padding.right}px`;

// ✅ Sử dụng hook
const { settingPageStyles, settingContentPageStyles } = useConvertPageSettings({ pageSettings });
```

### RULE-CPS-05: Áp dụng styles trả về đúng vị trí — `settingPageStyles` cho container ngoài, `settingContentPageStyles` cho content bên trong

```tsx
// ✅ Áp dụng đúng vị trí
<div style={settingPageStyles}>
  <div style={settingContentPageStyles}>
    {children}
  </div>
</div>
```
