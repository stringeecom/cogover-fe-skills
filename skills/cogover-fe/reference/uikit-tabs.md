## Tabs Component (`uikit-tabs`)

Đường dẫn: `ui-kit/src/components/Tabs`

Props: `tabs` (bắt buộc), `currentTabIndex`, `onChange`, `renderRightIcon`, `scrollable`, `underline`, `layout`, `className`.

---

### RULE-TABS-01: `Tabs` không có state nội bộ — luôn truyền `currentTabIndex`

```tsx
// ❌ Không tab nào được highlight
<Tabs tabs={["Overview", "Settings", "Logs"]} />

// ✅ Với onChange — xử lý click
const [activeTab, setActiveTab] = useState(0);
<Tabs tabs={["Overview", "Settings", "Logs"]} currentTabIndex={activeTab} onChange={setActiveTab} />

// ✅ Không có onChange — tab điều khiển bởi cơ chế khác
<Tabs tabs={["Overview", "Settings", "Logs"]} currentTabIndex={currentStep} />
```

---

### RULE-TABS-02: `currentTabIndex` là 0-based

```tsx
// ❌ Tab thứ hai được highlight thay vì tab đầu
<Tabs tabs={["Tab 1", "Tab 2", "Tab 3"]} currentTabIndex={1} onChange={setActiveTab} />

// ✅
<Tabs tabs={["Tab 1", "Tab 2", "Tab 3"]} currentTabIndex={0} onChange={setActiveTab} />
```

---

### RULE-TABS-03: Dùng `scrollable` khi tabs có thể tràn container

```tsx
// ✅
<Tabs tabs={manyLongTabs} currentTabIndex={activeTab} onChange={setActiveTab} scrollable />
```

---

### RULE-TABS-04: Dùng `layout={TAB_LAYOUT.VERTICAL}` — không override flex direction qua className

```tsx
// ❌
<Tabs tabs={steps} currentTabIndex={0} onChange={setActiveTab} className="flex-col" />

// ✅
import { TAB_LAYOUT } from "ui-kit";
<Tabs tabs={steps} currentTabIndex={activeTab} onChange={setActiveTab} layout={TAB_LAYOUT.VERTICAL} />
```

---

### RULE-TABS-05: Dùng `renderRightIcon` cho actions bên phải tab bar — không wrap thủ công

```tsx
// ❌ Wrapper phá vỡ alignment với tab underline
<div className="flex items-center">
  <Tabs tabs={["Tab 1", "Tab 2"]} currentTabIndex={0} onChange={setActiveTab} />
  <AddButton />
</div>

// ✅
<Tabs
  tabs={["Tab 1", "Tab 2"]}
  currentTabIndex={activeTab}
  onChange={setActiveTab}
  renderRightIcon={() => <AddButton />}
/>
```
