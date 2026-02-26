---
title: Cách sử dụng component Tabs trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến trạng thái tab không được kiểm soát, scrolling bị lỗi hoặc hướng layout sai
tags: tabs, navigation, ui-kit
---

## Cách sử dụng component Tabs trong ui-kit

Đường dẫn component: `ui-kit/src/components/Tabs`

Props: `tabs` (bắt buộc), `currentTabIndex`, `onChange` (optional), `renderRightIcon`, `scrollable`, `underline`, `layout`, `className`.

---

### RULE-TABS-01: `Tabs` không có state nội bộ — luôn truyền `currentTabIndex` để hiển thị tab đang active

`Tabs` không có state nội bộ. Không có `currentTabIndex`, tab active không được highlight. `onChange` chỉ cần truyền khi muốn xử lý sự kiện click tab (ví dụ: chuyển tab). Nếu tab được điều khiển bởi cơ chế khác (URL routing, logic bên ngoài), có thể bỏ `onChange`.

**Sai:**

```tsx
// ❌ Không có currentTabIndex — không tab nào được highlight
<Tabs tabs={["Overview", "Settings", "Logs"]} />
```

**Đúng:**

```tsx
// ✅ Có onChange — dùng khi cần xử lý click chuyển tab
const [activeTab, setActiveTab] = useState(0);

<Tabs
  tabs={["Overview", "Settings", "Logs"]}
  currentTabIndex={activeTab}
  onChange={(index) => setActiveTab(index)}
/>;
```

```tsx
// ✅ Không có onChange — dùng khi tab được điều khiển bởi cơ chế khác
<Tabs
  tabs={["Overview", "Settings", "Logs"]}
  currentTabIndex={currentStep}
/>;
```

---

### RULE-TABS-02: `currentTabIndex` là 0-based — không bao giờ truyền giá trị 1-based

Tab đang active được xác định bởi `index === currentTabIndex`. Truyền `1` kích hoạt tab thứ hai, không phải tab đầu tiên.

**Sai:**

```tsx
// ❌ 1-based — tab thứ hai được highlight thay vì tab đầu tiên
<Tabs
  tabs={["Tab 1", "Tab 2", "Tab 3"]}
  currentTabIndex={1}
  onChange={setActiveTab}
/>
```

**Đúng:**

```tsx
// ✅ 0-based — tab đầu tiên được highlight đúng
<Tabs
  tabs={["Tab 1", "Tab 2", "Tab 3"]}
  currentTabIndex={0}
  onChange={setActiveTab}
/>
```

---

### RULE-TABS-03: Sử dụng `scrollable` khi tabs có thể tràn ra ngoài độ rộng container

`scrollable` set `overflow-x: auto` và cố định chiều cao container ở 38px. Không có nó, tabs wrap xuống dòng tiếp theo hoặc tràn rõ ràng.

**Sai:**

```tsx
// ❌ Nhiều tabs không có scrollable — tabs tràn hoặc wrap
<Tabs tabs={manyLongTabs} currentTabIndex={activeTab} onChange={setActiveTab} />
```

**Đúng:**

```tsx
// ✅ Bật scrollable khi số lượng hoặc độ rộng tabs biến đổi
<Tabs
  tabs={manyLongTabs}
  currentTabIndex={activeTab}
  onChange={setActiveTab}
  scrollable
/>
```

---

### RULE-TABS-04: Sử dụng `underline` để hiển thị baseline full-width dưới tất cả tabs

`underline` render một đường ngang `2px` kéo dài full-width phía sau tất cả các tab indicators. Sử dụng nó khi tab bar cần baseline để tách rời nó khỏi nội dung bên dưới một cách trực quan.

**Đúng:**

```tsx
// ✅ Với underline baseline
<Tabs
  tabs={["Tab 1", "Tab 2"]}
  currentTabIndex={activeTab}
  onChange={setActiveTab}
  underline
/>
```

---

### RULE-TABS-05: Sử dụng `layout={TAB_LAYOUT.VERTICAL}` cho danh sách tab dọc — không dùng CSS overrides

`layout` chuyển đổi flex direction của danh sách tab thành `flex-col`. Không sử dụng `className` để override flex direction thủ công.

**Sai:**

```tsx
// ❌ Override layout qua className — phá vỡ các styles nội bộ
<Tabs
  tabs={["Step 1", "Step 2"]}
  currentTabIndex={0}
  onChange={setActiveTab}
  className="flex-col"
/>
```

**Đúng:**

```tsx
import { TAB_LAYOUT } from "ui-kit";

// ✅ Sử dụng prop layout
<Tabs
  tabs={["Step 1", "Step 2"]}
  currentTabIndex={activeTab}
  onChange={setActiveTab}
  layout={TAB_LAYOUT.VERTICAL}
/>;
```

---

### RULE-TABS-06: Sử dụng `renderRightIcon` cho các hành động ở phía bên phải tab bar — không wrap thủ công

Component đặt right icon trong một container `absolute right-0 bottom-0`. Không wrap `Tabs` trong một flex div để đạt được layout này thủ công.

**Sai:**

```tsx
// ❌ Wrapper thủ công — alignment với tab underline bị lỗi
<div className="flex items-center">
  <Tabs tabs={["Tab 1", "Tab 2"]} currentTabIndex={0} onChange={setActiveTab} />
  <AddButton />
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng renderRightIcon — được đặt đúng tương đối với tab bar
<Tabs
  tabs={["Tab 1", "Tab 2"]}
  currentTabIndex={activeTab}
  onChange={setActiveTab}
  renderRightIcon={() => <AddButton />}
/>
```
