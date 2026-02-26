---
title: Cách sử dụng component UsageHelpButton trong ui-kit
impact: LOW
impactDescription: Sử dụng sai dẫn đến null renders hoặc sử dụng sai các help keys không xác định
tags: help, tooltip, documentation, ui-kit
---

## Cách sử dụng component UsageHelpButton trong ui-kit

Đường dẫn component: `ui-kit/src/components/UsageHelpButton`

Props: `helpKey` (HelpTextVariant), `mode` (icon/text), `textLinkProps`.

---

### RULE-USAGE-HELP-01: Chỉ sử dụng các keys định trước từ `HelpTextVariant` — không truyền các chuỗi tùy ý

`UsageHelpButton` map `helpKey` đến các bản dịch i18n trong namespace `usage_help` và đến một help URL qua `getHelpLink`. Truyền một key không có trong union `HelpTextVariant` sẽ gây lỗi TypeScript và render nội dung bị lỗi.

**Sai:**

```tsx
// ❌ Chuỗi tùy ý — lỗi TypeScript và nội dung bị lỗi
<UsageHelpButton helpKey="my_custom_help_text" mode="icon" />
```

**Đúng:**

```tsx
// ✅ Sử dụng một key từ union HelpTextVariant
<UsageHelpButton helpKey="filter" mode="icon" />
<UsageHelpButton helpKey="app_manager" mode="icon" />
<UsageHelpButton helpKey="cogover_scripting" mode="text" />
```

---

### RULE-USAGE-HELP-02: Sử dụng `mode="icon"` cho inline help bên cạnh field label — sử dụng `mode="text"` cho đoạn help mô tả

- `mode="icon"` wrap nội dung help bên trong một popover `HelpButton` (một icon `?` mở ra tooltip)
- `mode="text"` render help text inline như một `ReactNode` (không có icon, không có popover)

**Đúng:**

```tsx
// ✅ Bên cạnh form label — icon nhỏ gọn và không xâm lấn
<div className="flex items-center gap-1">
  <label>Filter Rules</label>
  <UsageHelpButton helpKey="filter" mode="icon" />
</div>

// ✅ Dưới section header — text render inline như một mô tả
<UsageHelpButton helpKey="app_manager" mode="text" />
```

---

### RULE-USAGE-HELP-03: Không render có điều kiện dựa trên `helpKey` — component tự trả về `null` khi `helpKey` là falsy

`UsageHelpButton` đã trả về `null` khi `helpKey` không được cung cấp. Wrap nó trong một conditional bổ sung là thừa.

**Sai:**

```tsx
// ❌ Conditional không cần thiết
{
  helpKey && <UsageHelpButton helpKey={helpKey} mode="icon" />;
}
```

**Đúng:**

```tsx
// ✅ Component tự xử lý null nội bộ
<UsageHelpButton helpKey={helpKey} mode="icon" />
```

---

### RULE-USAGE-HELP-04: Sử dụng `textLinkProps` để tùy chỉnh link bên trong nội dung help — không wrap component để chặn clicks link

Nội dung help được render bởi `UsageHelpButton` có thể chứa một component `TextLink` trỏ đến tài liệu bên ngoài. Sử dụng `textLinkProps` để tùy chỉnh appearance hoặc behavior của nó.

**Sai:**

```tsx
// ❌ Wrapping để chặn link — không hoạt động, link nằm nội bộ component
<div onClick={(e) => e.preventDefault()}>
  <UsageHelpButton helpKey="filter" mode="text" />
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng textLinkProps để tùy chỉnh link
<UsageHelpButton
  helpKey="filter"
  mode="text"
  textLinkProps={{ target: "_self", className: "text-sm" }}
/>
```
