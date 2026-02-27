## MonacoEditor Component (`uikit-monaco-editor`)

Đường dẫn: `ui-kit/src/components/MonacoEditor`

Wrapper của `@monaco-editor/react` với loader path đã cấu hình tới `/static/js/monaco-editor/min/vs`. Export thêm hook `useMonaco`.

Props: Tất cả `EditorProps` từ `@monaco-editor/react` — `value`, `defaultValue`, `language`, `theme`, `height`, `width`, `options`, `onChange`, `onMount`, `onValidate`, `beforeMount`, `defaultLanguage`, `path`, `line`, `loading`, `className`, `wrapperProps`, `keepCurrentModel`, `saveViewState`.

---

### RULE-MONACO-01: Import từ ui-kit — không import trực tiếp từ `@monaco-editor/react`

```tsx
// ❌ Dùng CDN mặc định, có thể bị chặn
import { Editor } from "@monaco-editor/react";

// ✅
import { MonacoEditor, useMonaco } from "ui-kit";
<MonacoEditor language="json" value={code} onChange={setCode} />
```

---

### RULE-MONACO-02: Không gọi lại `loader.config` — ui-kit đã cấu hình sẵn

```tsx
// ❌ Ghi đè cấu hình — gây lỗi load
import { loader } from "@monaco-editor/react";
loader.config({ paths: { vs: "https://cdn.jsdelivr.net/npm/monaco-editor/min/vs" } });

// ✅ Chỉ cần sử dụng
<MonacoEditor language="json" value={code} onChange={setCode} />
```

---

### RULE-MONACO-03: Cấu hình editor qua prop `options`

```tsx
// ✅
<MonacoEditor
  language="json"
  height={200}
  theme="vs-dark"
  options={{
    quickSuggestions: true,
    autoClosingBrackets: "always",
    fontSize: 14,
    padding: { top: 20, bottom: 20 },
    scrollBeyondLastLine: false,
  }}
/>
```

---

### RULE-MONACO-04: Dùng prop `height` — không override bằng CSS bên ngoài

```tsx
// ❌ Editor không tính đúng kích thước layout
<div style={{ height: 200 }}><MonacoEditor language="json" /></div>

// ✅
<MonacoEditor language="json" height={200} />
<MonacoEditor language="json" height="50vh" />
```

---

### RULE-MONACO-05: Dùng `useMonaco` để truy cập Monaco instance

```tsx
// ❌ Import trực tiếp — xung đột version, thiếu cấu hình loader
import * as monaco from "monaco-editor";

// ✅
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

### RULE-MONACO-06: Wrap trong container có chiều rộng xác định

```tsx
// ✅
<div className={cx("w-[31.25rem]")}>
  <MonacoEditor language="json" height={200} />
</div>

// ✅ Hoặc dùng prop width
<MonacoEditor language="json" height={200} width="100%" />
```
