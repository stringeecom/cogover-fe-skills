---
title: Cách sử dụng component LayoutSelectButton trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến nút chọn layout không hoạt động, layout không được chuyển đổi, hoặc giao diện không nhất quán khi chọn layout
tags: layout, select, button, popper, menu, ui-kit
---

## Cách sử dụng component LayoutSelectButton trong ui-kit

Đường dẫn component: `ui-kit/src/components/LayoutSelectButton`

Props bắt buộc: `listLayouts`, `activeLayoutId`, `onChangeLayoutId`.

Không có props tuỳ chọn. Component này là một nút bấm hiển thị danh sách layout trong Popper menu, cho phép người dùng chọn layout. Layout đang active sẽ được highlight và hiển thị icon check.

Kiểu dữ liệu `listLayouts`: `{ name: string; id: string }[]`.
Kiểu dữ liệu `activeLayoutId`: `string | null`.
Kiểu dữ liệu `onChangeLayoutId`: `(layoutId: string) => void`.

---

### RULE-LAYOUT-SELECT-BTN-01: LayoutSelectButton là controlled component — phải truyền đủ 3 props bắt buộc

`LayoutSelectButton` là controlled component. Phải truyền `listLayouts` (danh sách layout), `activeLayoutId` (id layout đang active), và `onChangeLayoutId` (callback khi chọn layout mới). Thiếu bất kỳ prop nào sẽ gây lỗi TypeScript hoặc component không hoạt động đúng.

**Sai:**

```tsx
// ❌ Thiếu onChangeLayoutId — không thể chuyển layout
<LayoutSelectButton
  listLayouts={layouts}
  activeLayoutId={activeId}
/>
```

**Đúng:**

```tsx
// ✅ Truyền đủ 3 props bắt buộc
<LayoutSelectButton
  listLayouts={layouts}
  activeLayoutId={activeId}
  onChangeLayoutId={setActiveId}
/>
```

---

### RULE-LAYOUT-SELECT-BTN-02: Sử dụng `useGetLayout` hook để lấy dữ liệu cho LayoutSelectButton — không tự build state và fetch layout

Hook `useGetLayout` đã cung cấp sẵn `listLayouts`, `activeLayoutId`, và `setActiveLayoutId` phù hợp với props của `LayoutSelectButton`. Không tự build state riêng để quản lý danh sách layout và layout đang active.

**Sai:**

```tsx
// ❌ Tự build state riêng — thiếu logic đồng bộ layout với URL, default layout, v.v.
const [layouts, setLayouts] = useState<{ name: string; id: string }[]>([]);
const [activeId, setActiveId] = useState<string | null>(null);

useEffect(() => {
  fetchLayouts().then(setLayouts);
}, []);

<LayoutSelectButton
  listLayouts={layouts}
  activeLayoutId={activeId}
  onChangeLayoutId={setActiveId}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng useGetLayout hook — đã xử lý sẵn logic fetch, default layout, URL sync
const { listLayouts, activeLayoutId, setActiveLayoutId } = useGetLayout({
  functionLabel: "ADD",
  objectSlug: "contact",
  saveToSearchParams: false,
});

<LayoutSelectButton
  listLayouts={listLayouts}
  activeLayoutId={activeLayoutId}
  onChangeLayoutId={setActiveLayoutId}
/>
```

---

### RULE-LAYOUT-SELECT-BTN-03: Không tự build dropdown chọn layout — sử dụng LayoutSelectButton

`LayoutSelectButton` đã tích hợp sẵn `Popper`, `PopperMenu`, `PopperMenuItem`, `IconButton`, `Tooltip` và logic highlight layout đang active với icon check. Không tự build giao diện tương tự bên ngoài.

**Sai:**

```tsx
// ❌ Tự build dropdown chọn layout — trùng lặp logic, thiếu UX nhất quán
<Popper
  render={({ attributes, ...params }) => (
    <PopperMenu {...attributes} {...params}>
      {layouts.map((layout) => (
        <PopperMenuItem key={layout.id} onClick={() => setActiveId(layout.id)}>
          {layout.name}
        </PopperMenuItem>
      ))}
    </PopperMenu>
  )}
>
  {(params) => (
    <IconButton {...params} variant="text" size="small">
      <FontAwesomeIcon icon={faEllipsisVertical} />
    </IconButton>
  )}
</Popper>
```

**Đúng:**

```tsx
// ✅ Sử dụng LayoutSelectButton — đã tích hợp sẵn tất cả UI và logic
<LayoutSelectButton
  listLayouts={layouts}
  activeLayoutId={activeId}
  onChangeLayoutId={setActiveId}
/>
```

---

### RULE-LAYOUT-SELECT-BTN-04: Đặt LayoutSelectButton trong container flex justify-end — theo chuẩn layout của dự án

Theo convention trong dự án, `LayoutSelectButton` được đặt trong một container flex với `justify-end` để nút nằm bên phải. Sử dụng `cx()` để nhóm các class Tailwind.

**Sai:**

```tsx
// ❌ Không có container — nút không được căn chỉnh đúng vị trí
<LayoutSelectButton
  listLayouts={listLayouts}
  activeLayoutId={activeLayoutId}
  onChangeLayoutId={setActiveLayoutId}
/>
```

**Đúng:**

```tsx
// ✅ Đặt trong container flex justify-end với cx()
<div className={cx("flex justify-end items-center gap-[0.5rem]", "mb-[1rem]")}>
  <LayoutSelectButton
    listLayouts={listLayouts}
    activeLayoutId={activeLayoutId}
    onChangeLayoutId={setActiveLayoutId}
  />
</div>
```

---

### RULE-LAYOUT-SELECT-BTN-05: Ẩn LayoutSelectButton khi danh sách layout rỗng hoặc đang loading — tránh hiển thị nút không có chức năng

Khi `listLayouts` rỗng hoặc dữ liệu đang loading, nút bấm sẽ mở Popper rỗng. Kiểm tra điều kiện trước khi render component.

**Sai:**

```tsx
// ❌ Luôn hiển thị — khi loading hoặc không có layout, nút bấm vô nghĩa
const { listLayouts, activeLayoutId, setActiveLayoutId, isLoading } = useGetLayout({
  functionLabel: "ADD",
  objectSlug: "contact",
});

<LayoutSelectButton
  listLayouts={listLayouts}
  activeLayoutId={activeLayoutId}
  onChangeLayoutId={setActiveLayoutId}
/>
```

**Đúng:**

```tsx
// ✅ Kiểm tra điều kiện trước khi render
const { listLayouts, activeLayoutId, setActiveLayoutId, isLoading } = useGetLayout({
  functionLabel: "ADD",
  objectSlug: "contact",
});

{!isLoading && listLayouts.length > 0 && (
  <LayoutSelectButton
    listLayouts={listLayouts}
    activeLayoutId={activeLayoutId}
    onChangeLayoutId={setActiveLayoutId}
  />
)}
```

---

### RULE-LAYOUT-SELECT-BTN-06: Mỗi item trong `listLayouts` phải có `id` và `name` duy nhất — không truyền dữ liệu trùng lặp hoặc thiếu

Component sử dụng `layout.id` làm `key` cho danh sách và so sánh với `activeLayoutId` để xác định layout đang active. Nếu `id` trùng lặp hoặc thiếu `name`, giao diện sẽ không hiển thị đúng.

**Sai:**

```tsx
// ❌ Thiếu name và id trùng lặp — giao diện hiển thị sai
const layouts = [
  { id: "1", name: "" },
  { id: "1", name: "Layout B" },
];

<LayoutSelectButton
  listLayouts={layouts}
  activeLayoutId={activeId}
  onChangeLayoutId={setActiveId}
/>
```

**Đúng:**

```tsx
// ✅ Mỗi layout có id duy nhất và name rõ ràng
const layouts = [
  { id: "layout-1", name: "Layout mặc định" },
  { id: "layout-2", name: "Layout chi tiết" },
];

<LayoutSelectButton
  listLayouts={layouts}
  activeLayoutId={activeId}
  onChangeLayoutId={setActiveId}
/>
```

---

### Bảng tham chiếu giá trị mặc định

| Thuộc tính nội bộ        | Giá trị mặc định        | Ghi chú                                                    |
| ------------------------- | ----------------------- | ----------------------------------------------------------- |
| Popper placement          | `bottom-end`            | Dropdown mở ở phía dưới bên phải nút                       |
| Popper offset             | `[0, 4]`                | Khoảng cách 4px giữa nút và dropdown                       |
| IconButton variant        | `text`                  | Nút trong suốt, không có nền                               |
| IconButton size           | `small`                 | Kích thước nhỏ                                              |
| Icon                      | `faEllipsisVertical`    | Icon ba chấm dọc (FontAwesome pro-regular)                  |
| Tooltip content           | `t("common:common.layout")` | Hiển thị tooltip "Layout" khi hover                    |
| Active item className     | `active !bg-primary-light-90` | Highlight layout đang active                          |
| Active item check icon    | `faCheck`               | Icon check hiển thị bên phải layout đang active             |
| PopperMenu className      | `hide-on-click`         | Tự động đóng dropdown khi click vào item                    |
