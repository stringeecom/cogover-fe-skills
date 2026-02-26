---
title: Cách sử dụng BreadCrumbs Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng BreadCrumbs sai dẫn đến việc điều hướng không chính xác, dropdown menu không hiển thị đúng và label bị cắt ngắn không mong muốn
tags: breadcrumbs, navigation, dropdown, link, ui-kit
---

## Cách sử dụng BreadCrumbs Component của ui-kit

Đường dẫn component: `ui-kit/src/components/BreadCrumbs`

Props: `items`, `isDisabled`, `isFullLabel`, `className`. Hỗ trợ tất cả `React.HTMLAttributes<HTMLDivElement>`.

Mỗi item trong `items` có type `BreadCrumbItems` với các thuộc tính: `label`, `href`, `isShow`, `isHide`, `isAutoRedirect`, `children`, `isActive`.

---

### RULE-BREADCRUMBS-01: Truyền `items` đúng cấu trúc `BreadCrumbItems[]`

Mỗi item bắt buộc phải có `label` (string). Các thuộc tính còn lại là optional. Khi item có `href`, nó sẽ tự động render thành `Link` có thể click để điều hướng. Item cuối cùng sẽ được render với variant `body1` (đậm hơn), các item còn lại dùng variant `body2`.

**Sai:**

```tsx
// ❌ Truyền items không có label
<BreadCrumbs items={[{ href: "/home" }, { href: "/products" }]} />
```

**Đúng:**

```tsx
// ✅ Mỗi item đều có label, href là optional
<BreadCrumbs
  items={[
    { label: "Trang chủ", href: "/home" },
    { label: "Sản phẩm", href: "/products" },
    { label: "Chi tiết sản phẩm" },
  ]}
/>
```

---

### RULE-BREADCRUMBS-02: Sử dụng `children` để tạo dropdown menu cho item

Khi một item có `children`, nó sẽ hiển thị icon mũi tên xuống và mở `Popper` dropdown khi click. Mỗi child item cũng có cấu trúc `BreadCrumbItems`. Sử dụng `isShow` trên child item để ẩn item đó trong dropdown. Sử dụng `isAutoRedirect` trên item cha để item cha vừa có dropdown vừa có thể click để điều hướng.

**Sai:**

```tsx
// ❌ Tự tạo dropdown bằng Popper bên ngoài BreadCrumbs
<div className={cx("flex items-center")}>
  <BreadCrumbs items={[{ label: "Danh mục" }]} />
  <Popper render={() => <PopperMenu>...</PopperMenu>}>
    {(params) => <button {...params}>▼</button>}
  </Popper>
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng children để tạo dropdown tự động
<BreadCrumbs
  items={[
    {
      label: "Danh mục",
      href: "/categories",
      isAutoRedirect: true,
      children: [
        { label: "Điện thoại", href: "/categories/phones" },
        { label: "Laptop", href: "/categories/laptops" },
        { label: "Phụ kiện", href: "/categories/accessories" },
      ],
    },
    { label: "Chi tiết" },
  ]}
/>
```

---

### RULE-BREADCRUMBS-03: Sử dụng `isAutoRedirect` khi item có `children` và cần click để điều hướng

Khi item có `children`, mặc định click vào label sẽ chỉ mở dropdown. Nếu muốn label vừa mở dropdown vừa có thể click để điều hướng đến `href`, phải truyền `isAutoRedirect: true`.

**Sai:**

```tsx
// ❌ Item có children và href nhưng thiếu isAutoRedirect — click label không điều hướng
<BreadCrumbs
  items={[
    {
      label: "Danh mục",
      href: "/categories",
      children: [
        { label: "Điện thoại", href: "/categories/phones" },
      ],
    },
  ]}
/>
```

**Đúng:**

```tsx
// ✅ Thêm isAutoRedirect để click label cũng điều hướng đến href
<BreadCrumbs
  items={[
    {
      label: "Danh mục",
      href: "/categories",
      isAutoRedirect: true,
      children: [
        { label: "Điện thoại", href: "/categories/phones" },
      ],
    },
  ]}
/>
```

---

### RULE-BREADCRUMBS-04: Sử dụng `isFullLabel` khi không muốn label bị cắt ngắn

Khi container bị tràn, BreadCrumbs tự động cắt ngắn label thành 30 ký tự đầu và thêm "..." phía sau (có tooltip hiển thị đầy đủ). Nếu muốn hiển thị toàn bộ label, truyền `isFullLabel`.

**Sai:**

```tsx
// ❌ Tự cắt ngắn label trước khi truyền vào
<BreadCrumbs
  items={[
    { label: "Danh mục sản phẩm điện tử và ph...".slice(0, 30) + "..." },
  ]}
/>
```

**Đúng:**

```tsx
// ✅ Để BreadCrumbs tự xử lý việc cắt ngắn
<BreadCrumbs
  items={[
    { label: "Danh mục sản phẩm điện tử và phụ kiện công nghệ" },
  ]}
/>

// ✅ Truyền isFullLabel khi muốn hiển thị toàn bộ label
<BreadCrumbs
  isFullLabel
  items={[
    { label: "Danh mục sản phẩm điện tử và phụ kiện công nghệ" },
  ]}
/>
```

---

### RULE-BREADCRUMBS-05: Sử dụng `isDisabled` để vô hiệu hóa điều hướng trên toàn bộ BreadCrumbs

Khi truyền `isDisabled`, tất cả các link trong dropdown children sẽ bị vô hiệu hóa (render thành `span` thay vì `Link`) và hiển thị với opacity giảm. Lưu ý: `isDisabled` chỉ ảnh hưởng đến các child items trong dropdown, không ảnh hưởng đến các item chính.

**Sai:**

```tsx
// ❌ Tự xử lý disabled bằng cách xóa href
<BreadCrumbs
  items={[
    {
      label: "Danh mục",
      children: [
        { label: "Điện thoại" }, // Xóa href để "disabled"
      ],
    },
  ]}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng isDisabled để vô hiệu hóa toàn bộ
<BreadCrumbs
  isDisabled
  items={[
    {
      label: "Danh mục",
      children: [
        { label: "Điện thoại", href: "/categories/phones" },
        { label: "Laptop", href: "/categories/laptops" },
      ],
    },
  ]}
/>
```

---

### RULE-BREADCRUMBS-06: Sử dụng `isShow` trên item để ẩn item, không dùng filter bên ngoài

Trong dropdown children, `isShow` được dùng để ẩn child item (thêm class `hidden`). Đối với item chính, khi `isShow` là falsy, item sẽ được render bình thường. Khi `isShow` là truthy, item chính sẽ không được render.

**Sai:**

```tsx
// ❌ Filter items bên ngoài component
<BreadCrumbs
  items={allItems.filter((item) => item.isVisible)}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng isShow để kiểm soát hiển thị
<BreadCrumbs
  items={[
    {
      label: "Danh mục",
      children: [
        { label: "Điện thoại", href: "/phones", isShow: false },
        { label: "Laptop", href: "/laptops" },
      ],
    },
    { label: "Chi tiết", isShow: false },
  ]}
/>
```

---

## Tham chiếu style

| Phần tử | Style |
|---------|-------|
| Container | `flex gap-[1rem] items-center text-gray-70` |
| Item cuối cùng | `Typography variant="body1"`, `text-gray-80` |
| Các item khác | `Typography variant="body2"`, `text-gray-70` |
| Hover trên link | `hover:underline hover:text-primary-main` |
| Dấu phân cách | `/` giữa các item |
| Dropdown arrow | `FontAwesomeIcon faAngleDown`, fontSize 12 |
| Label bị cắt ngắn | Hiển thị 30 ký tự đầu + "..." kèm Tooltip |
