## Cách sử dụng component LayoutSelectButton trong ui-kit

Đường dẫn: `ui-kit/src/components/LayoutSelectButton`

Props bắt buộc (tất cả đều required):
- `listLayouts`: `{ name: string; id: string }[]`
- `activeLayoutId`: `string | null`
- `onChangeLayoutId`: `(layoutId: string) => void`

Component tích hợp sẵn: Popper, PopperMenu, PopperMenuItem, IconButton (`faEllipsisVertical`), Tooltip ("Layout"), highlight active với icon check.

```tsx
// Lấy dữ liệu với useGetLayout hook (không tự build state/fetch)
const { listLayouts, activeLayoutId, setActiveLayoutId, isLoading } = useGetLayout({
  functionLabel: "ADD",
  objectSlug: "contact",
  saveToSearchParams: false,
});

// Ẩn khi loading hoặc danh sách rỗng, đặt trong container flex justify-end
{!isLoading && listLayouts.length > 0 && (
  <div className={cx("flex justify-end items-center gap-[0.5rem]", "mb-[1rem]")}>
    <LayoutSelectButton
      listLayouts={listLayouts}
      activeLayoutId={activeLayoutId}
      onChangeLayoutId={setActiveLayoutId}
    />
  </div>
)}
```

**DO:** Dùng `useGetLayout` hook để lấy dữ liệu. Đảm bảo mỗi item có `id` duy nhất và `name` rõ ràng. Ẩn khi `listLayouts.length === 0` hoặc đang loading. Đặt trong container `flex justify-end`.

**DON'T:** Tự build state/fetch layout. Tự build dropdown với Popper. Thiếu bất kỳ prop bắt buộc nào. Hiển thị khi danh sách rỗng hoặc đang loading. Dùng `id` trùng lặp hoặc `name` rỗng.

| Thuộc tính nội bộ | Giá trị |
|---|---|
| Popper placement | `bottom-end` |
| IconButton variant | `text`, size `small` |
| Active item | `active !bg-primary-light-90` + `faCheck` |
