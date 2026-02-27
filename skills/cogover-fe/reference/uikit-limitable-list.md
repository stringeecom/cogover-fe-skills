---
title: Cách sử dụng LimitableList Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng LimitableList sai dẫn đến việc tính toán chiều rộng không chính xác, hiển thị danh sách dư thừa và popup danh sách còn lại không đúng cách
tags: limitable-list, overflow, truncate, popper, tooltip, ui-kit
---

## Cách sử dụng LimitableList Component của ui-kit

Đường dẫn component: `ui-kit/src/components/LimitableList`

Props: `items`, `renderItem`, `getTooltipContent`, `getItemWidth`, `renderListItem`, `renderListItems`, `showSeparate`, `popperProps`.

Component này hiển thị một danh sách các phần tử trên một dòng, tự động tính toán số lượng phần tử có thể hiển thị dựa trên chiều rộng container. Các phần tử vượt quá sẽ được thu gọn thành nút `+N` và hiển thị trong popup khi click.

---

### RULE-LIMLIST-01: Cung cấp `getItemWidth` khi `renderItem` trả về nội dung có chiều rộng khác với text thuần

Mặc định, component tính chiều rộng mỗi phần tử bằng `getTextWidth(String(item))` với font-size 14px. Nếu `renderItem` render nội dung phức tạp (có icon, badge, padding...), bạn PHẢI cung cấp `getItemWidth` để tính toán chính xác.

**Sai:**

```tsx
// ❌ renderItem trả về tag có padding nhưng không cung cấp getItemWidth — tính toán chiều rộng sai
<LimitableList
  items={tags}
  renderItem={(tag) => (
    <span className={cx("px-[0.5rem] py-[0.125rem]", "bg-gray-5 rounded")}>
      {tag.name}
    </span>
  )}
/>
```

**Đúng:**

```tsx
// ✅ Cung cấp getItemWidth tương ứng với kích thước thực tế của renderItem
<LimitableList
  items={tags}
  renderItem={(tag) => (
    <span className={cx("px-[0.5rem] py-[0.125rem]", "bg-gray-5 rounded")}>
      {tag.name}
    </span>
  )}
  getItemWidth={(tag) => getTextWidth(tag.name, { fontSize: "14px" }) + 16}
/>
```

---

### RULE-LIMLIST-02: Sử dụng `renderItem` để tuỳ chỉnh hiển thị từng phần tử trong dòng chính, sử dụng `renderListItem` để tuỳ chỉnh hiển thị từng phần tử trong popup

`renderItem` quyết định cách hiển thị mỗi phần tử trên dòng chính (có SmartTooltip bao bọc). `renderListItem` quyết định cách hiển thị mỗi phần tử trong danh sách popup khi click nút `+N`. Không nhầm lẫn hai hàm này.

**Sai:**

```tsx
// ❌ Chỉ tuỳ chỉnh renderItem — popup vẫn hiển thị String(item) mặc định
<LimitableList
  items={users}
  renderItem={(user) => user.fullName}
/>
```

**Đúng:**

```tsx
// ✅ Tuỳ chỉnh cả renderItem và renderListItem khi item không phải kiểu string
<LimitableList
  items={users}
  renderItem={(user) => user.fullName}
  renderListItem={(user) => user.fullName}
  getTooltipContent={(user) => user.fullName}
  getItemWidth={(user) => getTextWidth(user.fullName, { fontSize: "14px" })}
/>
```

---

### RULE-LIMLIST-03: Truyền `showSeparate={false}` khi không muốn hiển thị dấu chấm phân cách giữa các phần tử

Mặc định `showSeparate` là `true`, hiển thị dấu chấm tròn (DotIcon) giữa các phần tử. Khi danh sách đã có phân cách riêng (ví dụ: tag có border, badge) thì nên tắt dấu chấm phân cách.

**Sai:**

```tsx
// ❌ Các tag đã có viền riêng, dấu chấm phân cách là dư thừa
<LimitableList
  items={tags}
  renderItem={(tag) => (
    <span className={cx("px-[0.5rem]", "border border-divider-primary rounded")}>
      {tag.name}
    </span>
  )}
/>
```

**Đúng:**

```tsx
// ✅ Tắt dấu chấm phân cách khi các phần tử đã có phân cách trực quan
<LimitableList
  showSeparate={false}
  items={tags}
  renderItem={(tag) => (
    <span className={cx("px-[0.5rem]", "border border-divider-primary rounded")}>
      {tag.name}
    </span>
  )}
/>
```

---

### RULE-LIMLIST-04: Sử dụng `renderListItems` khi cần tuỳ chỉnh toàn bộ nội dung popup, không override từng `renderListItem`

`renderListItems` nhận toàn bộ danh sách các phần tử còn lại và số lượng phần tử đang hiển thị, cho phép tuỳ chỉnh hoàn toàn layout popup. Chỉ sử dụng khi cần layout đặc biệt (ví dụ: table, grid). Trong trường hợp chỉ cần thay đổi nội dung từng dòng, sử dụng `renderListItem`.

**Sai:**

```tsx
// ❌ Dùng renderListItems chỉ để thay đổi nội dung từng dòng — quá phức tạp
<LimitableList
  items={users}
  renderListItems={(items) => (
    <ul className={cx("list-disc", "pl-[1.5rem]")}>
      {items.map((user, index) => (
        <li key={index}>{user.fullName}</li>
      ))}
    </ul>
  )}
/>
```

**Đúng:**

```tsx
// ✅ Dùng renderListItem cho tuỳ chỉnh đơn giản từng dòng
<LimitableList
  items={users}
  renderListItem={(user) => user.fullName}
/>

// ✅ Dùng renderListItems khi cần layout đặc biệt hoàn toàn khác mặc định
<LimitableList
  items={users}
  renderListItems={(items, displayedCount) => (
    <div className={cx("grid grid-cols-2", "gap-[0.5rem]")}>
      {items.map((user, index) => (
        <UserCard key={index} user={user} />
      ))}
    </div>
  )}
/>
```

---

### RULE-LIMLIST-05: Sử dụng `popperProps` để tuỳ chỉnh popup, không wrap LimitableList bằng Popper

Component đã tích hợp sẵn `Popper` cho nút `+N`. Sử dụng prop `popperProps` để tuỳ chỉnh vị trí, offset và các thuộc tính khác của popup.

**Sai:**

```tsx
// ❌ Wrap thêm Popper bên ngoài là dư thừa — component đã có Popper tích hợp
<Popper placement="bottom-end" render={({ attributes, ...params }) => (
  <div {...attributes} {...params}>
    <LimitableList items={items} />
  </div>
)}>
  {(params) => <div {...params}>Trigger</div>}
</Popper>
```

**Đúng:**

```tsx
// ✅ Tuỳ chỉnh Popper qua popperProps
<LimitableList
  items={items}
  popperProps={{
    placement: "bottom-end",
    offset: [0, 8],
  }}
/>
```

---

### RULE-LIMLIST-06: Cung cấp `getTooltipContent` khi item không phải kiểu string

Mặc định, tooltip hiển thị `String(item)`. Khi item là object, bạn PHẢI cung cấp `getTooltipContent` để hiển thị nội dung tooltip có ý nghĩa.

**Sai:**

```tsx
// ❌ item là object — tooltip sẽ hiển thị "[object Object]"
<LimitableList
  items={users}
  renderItem={(user) => user.fullName}
/>
```

**Đúng:**

```tsx
// ✅ Cung cấp getTooltipContent cho item dạng object
<LimitableList
  items={users}
  renderItem={(user) => user.fullName}
  getTooltipContent={(user) => user.fullName}
/>
```

---

## Tham chiếu callback

| Callback | Mục đích | Giá trị mặc định |
|----------|---------|-------------------|
| `renderItem` | Render phần tử trên dòng chính | `String(item)` |
| `renderListItem` | Render phần tử trong danh sách popup | `String(item)` |
| `renderListItems` | Render toàn bộ nội dung popup | Danh sách `<ul>` với `list-disc` |
| `getItemWidth` | Tính chiều rộng phần tử | `getTextWidth(String(item), { fontSize: "14px", fontWeight: 400 })` |
| `getTooltipContent` | Nội dung tooltip cho phần tử | `String(item)` |

## Tham chiếu props

| Prop | Kiểu | Mặc định | Mô tả |
|------|------|----------|-------|
| `items` | `T[]` | (bắt buộc) | Danh sách các phần tử cần hiển thị |
| `showSeparate` | `boolean` | `true` | Hiển thị dấu chấm phân cách giữa các phần tử |
| `popperProps` | `Partial<PopperProps>` | `undefined` | Tuỳ chỉnh thuộc tính của Popper popup |
