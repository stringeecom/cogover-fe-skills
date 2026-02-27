## Cách sử dụng BreadCrumbs Component của ui-kit

Đường dẫn: `ui-kit/src/components/BreadCrumbs`

Props: `items` (BreadCrumbItems[]), `isDisabled`, `isFullLabel`, `className`. Hỗ trợ `React.HTMLAttributes<HTMLDivElement>`.

`BreadCrumbItems`: `label` (bắt buộc), `href`, `isShow`, `isHide`, `isAutoRedirect`, `children` (BreadCrumbItems[]), `isActive`.

- Item có `href` → render thành `Link`
- Item có `children` → hiển thị dropdown Popper khi click
- Item cuối cùng → variant `body1` (đậm hơn)
- Label mặc định bị cắt ngắn 30 ký tự + tooltip

```tsx
// Cơ bản
<BreadCrumbs
  items={[
    { label: "Trang chủ", href: "/home" },
    { label: "Sản phẩm", href: "/products" },
    { label: "Chi tiết sản phẩm" },  // item cuối không cần href
  ]}
/>

// Với dropdown và isAutoRedirect (click label vừa navigate vừa mở dropdown)
<BreadCrumbs
  items={[
    {
      label: "Danh mục",
      href: "/categories",
      isAutoRedirect: true,       // không có → click chỉ mở dropdown, không navigate
      children: [
        { label: "Điện thoại", href: "/categories/phones" },
        { label: "Laptop", href: "/categories/laptops", isShow: false }, // ẩn trong dropdown
      ],
    },
    { label: "Chi tiết" },
  ]}
/>

// isFullLabel — hiển thị toàn bộ label không cắt ngắn
<BreadCrumbs isFullLabel items={[{ label: "Tên rất dài không muốn bị cắt ngắn..." }]} />

// isDisabled — vô hiệu hóa link trong dropdown children
<BreadCrumbs isDisabled items={[{ label: "Danh mục", children: [{ label: "Điện thoại", href: "/phones" }] }]} />
```

**DO:** Luôn truyền `label` cho mỗi item. Dùng `children` để tạo dropdown (không tự build Popper). Dùng `isAutoRedirect` khi item có cả `children` lẫn `href` và muốn click để navigate. Dùng `isFullLabel` thay vì tự cắt ngắn label. Dùng `isShow` trên child item để ẩn trong dropdown.

**DON'T:** Truyền item không có `label`. Tự build dropdown bên ngoài. Filter items bên ngoài thay vì dùng `isShow`. Tự cắt ngắn label trước khi truyền vào.
