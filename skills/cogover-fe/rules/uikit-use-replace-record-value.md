---
title: Cách sử dụng hook useReplaceRecordValue trong ui-kit
impact: HIGH
impactDescription: Hook thay thế template variables bằng giá trị record thực tế — tự fetch và replace sẽ thiếu field rendering và batch optimization
tags: hook, template, variable, record, replace, render, api, ui-kit
---

## Cách sử dụng hook useReplaceRecordValue trong ui-kit

Đường dẫn hook: `ui-kit/src/hooks/useReplaceRecordValue`

Hook thay thế template variables (dạng `{$variableName.path.to.field}`) bằng giá trị thực tế từ record, tự động fetch dữ liệu và render giá trị theo kiểu field.

### Giá trị trả về

| Giá trị | Kiểu | Mô tả |
| --- | --- | --- |
| `replaceRecordValue` | `(params: ReplaceRecordValueParams) => Promise<string>` | Hàm async thay thế variables |

### Params

| Param | Kiểu | Mô tả |
| --- | --- | --- |
| `template` | `string` | Chuỗi template chứa variables dạng `{$var.path}` |
| `variableRecordMap` | `Record<string, { objectSlug: string; recordId: string }>` | Map biến → record info |

### RULE-RRV-01: Template variables có format `{$variableName.path.to.field}` — `$` là bắt buộc

```tsx
const { replaceRecordValue } = useReplaceRecordValue();

const result = await replaceRecordValue({
    template: "Xin chào {$contact.name}, email: {$contact.email}",
    variableRecordMap: {
        "$contact": { objectSlug: "contacts", recordId: "123" },
    },
});
// result = "Xin chào Nguyễn Văn A, email: a@example.com"
```

### RULE-RRV-02: `variableRecordMap` key phải khớp với tên biến trong template (bao gồm `$`)

```tsx
// Template: "Giao cho {$user.name} thuộc {$department.name}"
const result = await replaceRecordValue({
    template: "Giao cho {$user.name} thuộc {$department.name}",
    variableRecordMap: {
        "$user": { objectSlug: "users", recordId: "u1" },
        "$department": { objectSlug: "departments", recordId: "d1" },
    },
});
```

### RULE-RRV-03: Hàm là async — phải await hoặc .then()

```tsx
// ✅ Await
const result = await replaceRecordValue({ template, variableRecordMap });

// ✅ Promise chain
replaceRecordValue({ template, variableRecordMap }).then(setDisplayText);
```

### RULE-RRV-04: Hook tự batch API calls và cache kết quả — không cần tối ưu bên ngoài

Hook nhóm variables theo recordId để giảm số lượng API calls, và sử dụng `queryClient.fetchQuery` cho caching.

### RULE-RRV-05: Không tự fetch record values và thay thế string — hook xử lý cả field metadata để render giá trị đúng format

```tsx
// ❌ Tự xử lý — thiếu field type rendering (date, currency, lookup...)
const record = await fetchRecord(recordId);
const result = template.replace("{$contact.name}", record.name);

// ✅ Hook render giá trị theo đúng kiểu field
const result = await replaceRecordValue({ template, variableRecordMap });
```
