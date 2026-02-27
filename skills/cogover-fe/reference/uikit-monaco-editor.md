---
title: Cách sử dụng component MonacoEditor trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến lỗi load editor, cấu hình loader trùng lặp, và mất tính năng IntelliSense
tags: monaco-editor, code-editor, syntax-highlighting, json-editor, ui-kit
---

## Cách sử dụng component MonacoEditor trong ui-kit

Đường dẫn component: `ui-kit/src/components/MonacoEditor`

Component `MonacoEditor` là một wrapper của `@monaco-editor/react` với loader path đã được cấu hình sẵn tới `/static/js/monaco-editor/min/vs`. Ngoài ra, ui-kit cũng export hook `useMonaco` để truy cập instance Monaco.

Props: Nhận tất cả props từ `EditorProps` của `@monaco-editor/react`, bao gồm: `value`, `defaultValue`, `language`, `theme`, `height`, `width`, `options`, `onChange`, `onMount`, `onValidate`, `beforeMount`, `defaultLanguage`, `path`, `line`, `loading`, `className`, `wrapperProps`, `keepCurrentModel`, `saveViewState`.

Hook: `useMonaco()` — trả về instance `Monaco | null`.

---

### RULE-MONACO-01: Import `MonacoEditor` và `useMonaco` từ ui-kit — không import trực tiếp từ `@monaco-editor/react`

ui-kit đã cấu hình `loader.config` với đường dẫn static assets phù hợp. Import trực tiếp từ `@monaco-editor/react` sẽ dùng CDN mặc định, có thể bị chặn hoặc tải chậm trong môi trường nội bộ.

**Sai:**

```tsx
// ❌ Import trực tiếp — không có cấu hình loader path của ui-kit
import { Editor } from "@monaco-editor/react";
import { useMonaco } from "@monaco-editor/react";

<Editor language="json" value={code} onChange={setCode} />
```

**Đúng:**

```tsx
// ✅ Import từ ui-kit — loader path đã được cấu hình sẵn
import { MonacoEditor, useMonaco } from "ui-kit";

<MonacoEditor language="json" value={code} onChange={setCode} />
```

---

### RULE-MONACO-02: Không gọi lại `loader.config` — ui-kit đã cấu hình sẵn

Component `MonacoEditor` và hook `useMonaco` đã gọi `loader.config({ paths: { vs: "/static/js/monaco-editor/min/vs" } })` khi import. Gọi lại sẽ ghi đè cấu hình và có thể gây lỗi load editor.

**Sai:**

```tsx
// ❌ Ghi đè cấu hình loader — có thể gây lỗi load
import { loader } from "@monaco-editor/react";
loader.config({ paths: { vs: "https://cdn.jsdelivr.net/npm/monaco-editor/min/vs" } });

<MonacoEditor language="json" value={code} onChange={setCode} />
```

**Đúng:**

```tsx
// ✅ Chỉ cần sử dụng, không cần cấu hình thêm
<MonacoEditor language="json" value={code} onChange={setCode} />
```

---

### RULE-MONACO-03: Truyền cấu hình editor qua prop `options` — không thao tác DOM trực tiếp

Sử dụng prop `options` để cấu hình các tính năng như gợi ý code, font size, padding, auto-closing brackets. Không tự truy cập DOM hoặc gọi API monaco trực tiếp để thay đổi cấu hình.

**Sai:**

```tsx
// ❌ Thao tác DOM trực tiếp — không đồng bộ với React lifecycle
const editorRef = useRef(null);
useEffect(() => {
  editorRef.current?.updateOptions({ fontSize: 14 });
}, []);

<MonacoEditor onMount={(editor) => { editorRef.current = editor; }} />
```

**Đúng:**

```tsx
// ✅ Cấu hình qua prop options
<MonacoEditor
  language="json"
  options={{
    quickSuggestions: true,
    suggestOnTriggerCharacters: true,
    autoClosingBrackets: "always",
    tabCompletion: "on",
    fontSize: 14,
    padding: {
      top: 20,
      bottom: 20,
    },
    scrollBeyondLastLine: false,
  }}
  height={200}
  theme="vs-dark"
/>
```

---

### RULE-MONACO-04: Sử dụng prop `height` để đặt chiều cao — không override bằng CSS bên ngoài

`MonacoEditor` nhận prop `height` (number hoặc string). Sử dụng prop này thay vì tự viết CSS để điều chỉnh chiều cao, vì editor cần biết kích thước chính xác để render đúng.

**Sai:**

```tsx
// ❌ Override CSS — editor có thể không tính đúng kích thước layout
<div style={{ height: 200 }}>
  <MonacoEditor language="json" />
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng prop height
<MonacoEditor language="json" height={200} />

// ✅ Hoặc dùng string
<MonacoEditor language="json" height="50vh" />
```

---

### RULE-MONACO-05: Sử dụng hook `useMonaco` để truy cập Monaco instance — không import trực tiếp `monaco-editor`

Khi cần truy cập Monaco instance (ví dụ: đăng ký language, theme, hoặc completion provider), sử dụng hook `useMonaco` từ ui-kit.

**Sai:**

```tsx
// ❌ Import trực tiếp — có thể xung đột version và thiếu cấu hình loader
import * as monaco from "monaco-editor";

monaco.languages.registerCompletionItemProvider("json", { ... });
```

**Đúng:**

```tsx
// ✅ Sử dụng hook useMonaco từ ui-kit
import { useMonaco } from "ui-kit";

const monaco = useMonaco();

useEffect(() => {
  if (monaco) {
    monaco.languages.registerCompletionItemProvider("json", {
      provideCompletionItems: () => ({ suggestions: [] }),
    });
  }
}, [monaco]);
```

---

### RULE-MONACO-06: Wrap `MonacoEditor` trong container có chiều rộng xác định

`MonacoEditor` sẽ tự mở rộng theo chiều rộng của container cha. Nếu container cha không có chiều rộng xác định, editor có thể bị tràn hoặc không hiển thị đúng.

**Sai:**

```tsx
// ❌ Không có container — editor có thể bị tràn layout
<MonacoEditor language="json" height={200} />
```

**Đúng:**

```tsx
// ✅ Wrap trong container có chiều rộng xác định
<div className={cx("w-[31.25rem]")}>
  <MonacoEditor language="json" height={200} />
</div>

// ✅ Hoặc sử dụng prop width
<MonacoEditor language="json" height={200} width="100%" />
```
