## Cách sử dụng LimitableList Component của ui-kit

Đường dẫn: `ui-kit/src/components/LimitableList`

Props: `items` (T[], bắt buộc), `renderItem`, `getTooltipContent`, `getItemWidth`, `renderListItem`, `renderListItems`, `showSeparate` (default true), `popperProps`.

Tự tính số phần tử hiển thị dựa trên chiều rộng container. Phần tử vượt quá → nút `+N` → click mở Popper popup.

```tsx
// Chuỗi đơn giản (mặc định tính width theo text)
<LimitableList items={["Tag A", "Tag B", "Tag C"]} />

// Object — cần getTooltipContent, renderItem, renderListItem, getItemWidth
<LimitableList
  items={users}
  renderItem={(user) => user.fullName}
  renderListItem={(user) => user.fullName}
  getTooltipContent={(user) => user.fullName}  // mặc định String(item) → "[object Object]" nếu không có
  getItemWidth={(user) => getTextWidth(user.fullName, { fontSize: "14px" })}
/>

// Tag có padding — cần getItemWidth bù thêm padding
<LimitableList
  showSeparate={false}              // tắt dấu chấm khi item đã có border riêng
  items={tags}
  renderItem={(tag) => (
    <span className={cx("px-[0.5rem] py-[0.125rem]", "bg-gray-5 rounded")}>{tag.name}</span>
  )}
  getItemWidth={(tag) => getTextWidth(tag.name, { fontSize: "14px" }) + 16}  // +16 cho padding
/>

// Tuỳ chỉnh toàn bộ popup layout (dùng renderListItems cho layout đặc biệt)
<LimitableList
  items={users}
  renderListItems={(items) => (
    <div className={cx("grid grid-cols-2", "gap-[0.5rem]")}>
      {items.map((user, i) => <UserCard key={i} user={user} />)}
    </div>
  )}
  popperProps={{ placement: "bottom-end", offset: [0, 8] }}
/>
```

**DO:** Cung cấp `getItemWidth` khi `renderItem` trả về nội dung có padding/icon. Cung cấp `getTooltipContent` khi item là object. Dùng `renderListItem` cho tuỳ chỉnh từng dòng trong popup. Dùng `renderListItems` chỉ khi cần layout đặc biệt. Dùng `popperProps` để tuỳ chỉnh popup. Dùng `showSeparate={false}` khi item đã có phân cách trực quan.

**DON'T:** Bỏ `getItemWidth` khi item phức tạp (tính width sai). Dùng item object mà không có `getTooltipContent` (sẽ hiện `[object Object]`). Wrap LimitableList bằng Popper bên ngoài.
