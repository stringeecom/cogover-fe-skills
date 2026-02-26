---
title: Cách sử dụng TagsInput Component của ui-kit
impact: HIGH
impactDescription: Sử dụng TagsInput sai dẫn đến mất dữ liệu khi submit, suggest list không hoạt động, và tag không hiển thị đúng
tags: tags-input, tag, input, suggest, autocomplete, ui-kit
---

## Cách sử dụng TagsInput Component của ui-kit

Đường dẫn component: `ui-kit/src/components/TagsInput`

Props: `value`, `onChange`, `defaultValue`, `fullWidth`, `error`, `helpText`, `color`, `tagVariants`, `tagProps`, `placeholder`, `clearable`, `onClearable`, `submitWithStringValue`, `isChangeOverlap`, `startAdornment`, `endAdornment`, `searchable`, `filterBySearch`, `suggestListOptions`, `renderSuggestItem`, `renderTag`, `renderLimitTag`, `renderText`, `getLabel`, `getValue`, `getOptionWidth`, `getInputRef`, `onInputChange`, `handleSuggestOptionClickExtend`, `handleDeleteOptionExtend`, `wrapperClassName`, `popperClassName`, `appendToBody`, `tabIndex`. Hỗ trợ tất cả `React.InputHTMLAttributes` (trừ `value`, `onChange`, `defaultValue`).

Component hỗ trợ generic type `TagsInput<T>` cho phép sử dụng với kiểu dữ liệu tuỳ chỉnh ngoài `string`.

---

### RULE-TAGSINPUT-01: Khi sử dụng với object, phải truyền `getLabel` và `getValue`

Khi `T` không phải `string`, component cần biết cách lấy label và value từ object. Mặc định `getLabel` và `getValue` ép kiểu sang `string`, sẽ gây lỗi khi dữ liệu là object.

**Sai:**

```tsx
// ❌ Object được ép thành string — hiển thị "[object Object]"
<TagsInput<User>
  value={selectedUsers}
  onChange={setSelectedUsers}
/>
```

**Đúng:**

```tsx
// ✅ Truyền getLabel và getValue để component hiểu cách xử lý object
<TagsInput<User>
  value={selectedUsers}
  onChange={setSelectedUsers}
  getLabel={(user) => user.name}
  getValue={(user) => user.id}
/>
```

---

### RULE-TAGSINPUT-02: Sử dụng `submitWithStringValue={false}` khi chỉ cho phép chọn từ suggest list

Mặc định `submitWithStringValue` là `true`, nghĩa là khi người dùng nhấn Enter hoặc blur, giá trị trong input sẽ được thêm vào danh sách tag. Nếu bạn chỉ muốn người dùng chọn từ `suggestListOptions`, phải set `submitWithStringValue={false}`.

**Sai:**

```tsx
// ❌ Người dùng có thể nhập bất kỳ text nào và nhấn Enter để thêm tag — không mong muốn khi có suggest list
<TagsInput
  suggestListOptions={options}
  renderSuggestItem={(option) => <span>{option}</span>}
/>
```

**Đúng:**

```tsx
// ✅ Chỉ cho phép chọn từ suggest list, không cho phép nhập tự do
<TagsInput
  submitWithStringValue={false}
  suggestListOptions={options}
  renderSuggestItem={(option) => <span>{option}</span>}
/>
```

---

### RULE-TAGSINPUT-03: Bật `searchable` và `filterBySearch` đồng thời để lọc suggest list theo input

`searchable` và `filterBySearch` phải được bật cả hai để suggest list tự động lọc theo nội dung người dùng nhập. Chỉ bật `searchable` mà không bật `filterBySearch` sẽ không lọc danh sách.

**Sai:**

```tsx
// ❌ Chỉ bật searchable — danh sách suggest không được lọc
<TagsInput
  searchable
  suggestListOptions={options}
/>

// ❌ Chỉ bật filterBySearch — không có tác dụng vì thiếu searchable
<TagsInput
  filterBySearch
  suggestListOptions={options}
/>
```

**Đúng:**

```tsx
// ✅ Bật cả hai để lọc suggest list theo input
<TagsInput
  searchable
  filterBySearch
  suggestListOptions={options}
  renderSuggestItem={(option) => <span>{option}</span>}
/>
```

---

### RULE-TAGSINPUT-04: Sử dụng `renderSuggestItem` khi dùng `suggestListOptions` với kiểu object

Khi `suggestListOptions` chứa object (không phải string), phải truyền `renderSuggestItem` để hiển thị đúng. Mặc định component render trực tiếp item vào `PopperMenuItem`, sẽ hiển thị `[object Object]` với kiểu object.

**Sai:**

```tsx
// ❌ Object được render trực tiếp — hiển thị "[object Object]"
<TagsInput<User>
  suggestListOptions={users}
  getLabel={(user) => user.name}
  getValue={(user) => user.id}
/>
```

**Đúng:**

```tsx
// ✅ Truyền renderSuggestItem để hiển thị đúng
<TagsInput<User>
  suggestListOptions={users}
  getLabel={(user) => user.name}
  getValue={(user) => user.id}
  renderSuggestItem={(user) => (
    <PopperMenuItem>{user.name}</PopperMenuItem>
  )}
/>
```

---

### RULE-TAGSINPUT-05: Sử dụng `clearable` kết hợp với `onClearable` để xử lý logic khi xoá tất cả tag

Prop `clearable` hiển thị nút xoá (icon X) khi có ít nhất 1 tag. `onClearable` là callback nhận giá trị hiện tại trước khi xoá, dùng để xử lý side effect (VD: gọi API, cập nhật state khác).

**Sai:**

```tsx
// ❌ Bật clearable nhưng không xử lý side effect khi xoá
<TagsInput
  clearable
  value={tags}
  onChange={setTags}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng onClearable để xử lý logic trước khi xoá tất cả
<TagsInput
  clearable
  onClearable={(currentValues) => {
    // Xử lý side effect trước khi xoá
    trackEvent("tags_cleared", { count: currentValues.length });
  }}
  value={tags}
  onChange={setTags}
/>
```

---

### RULE-TAGSINPUT-06: Sử dụng `isChangeOverlap` để cho phép tag trùng lặp

Mặc định component loại bỏ các tag trùng lặp (dựa trên `getValue`). Nếu muốn cho phép tag trùng lặp, set `isChangeOverlap={true}`.

**Sai:**

```tsx
// ❌ Muốn cho phép trùng lặp nhưng không bật isChangeOverlap — tag trùng lặp bị loại bỏ
<TagsInput
  value={tags}
  onChange={setTags}
/>
```

**Đúng:**

```tsx
// ✅ Bật isChangeOverlap để cho phép tag trùng lặp
<TagsInput
  isChangeOverlap
  value={tags}
  onChange={setTags}
/>
```

---

### RULE-TAGSINPUT-07: Sử dụng `renderTag` để tuỳ chỉnh giao diện tag — không override qua `tagProps` className

`renderTag` cho phép tuỳ chỉnh toàn bộ giao diện của mỗi tag, nhận params đầy đủ (`value`, `index`, `isFocusedTag`, `isFocusedInput`, `disabled`, `onDelete`, `displayValueNumber`, `selectedOptions`). Override qua `tagProps` className có thể phá vỡ logic focus và delete.

**Sai:**

```tsx
// ❌ Override className qua tagProps — có thể phá vỡ styling focus và delete
<TagsInput
  tagProps={{ className: "bg-red-500 text-white" }}
  value={tags}
  onChange={setTags}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng renderTag để tuỳ chỉnh toàn bộ
<TagsInput
  renderTag={({ value, onDelete, isFocusedTag, disabled }) => (
    <Tag
      variant="outlined"
      isFocused={isFocusedTag}
      onDelete={!disabled ? onDelete : undefined}
      showDeleteIcon
    >
      {getLabel(value)}
    </Tag>
  )}
  value={tags}
  onChange={setTags}
/>
```

---

### RULE-TAGSINPUT-08: Truyền `startAdornment`/`endAdornment` đúng cách — hỗ trợ ReactNode hoặc function

`startAdornment` và `endAdornment` chấp nhận `React.ReactNode` hoặc `() => React.ReactNode`. Khi sử dụng `clearable`, `endAdornment` sẽ bị ẩn khi input đang focus.

**Sai:**

```tsx
// ❌ Truyền string thay vì ReactNode
<TagsInput
  startAdornment="icon"
  value={tags}
  onChange={setTags}
/>
```

**Đúng:**

```tsx
// ✅ Truyền ReactNode hoặc function
<TagsInput
  startAdornment={<SearchIcon />}
  endAdornment={() => <ChevronDownIcon />}
  value={tags}
  onChange={setTags}
/>
```

---

### RULE-TAGSINPUT-09: `error` tự động set `color="error"` — không cần truyền cả hai

Khi truyền `error={true}`, component tự động set `color` thành `"error"` cho `HelperText`. Không cần truyền `color="error"` riêng.

**Sai:**

```tsx
// ❌ Dư thừa — error đã tự động set color
<TagsInput
  error
  color="error"
  helpText="Trường này bắt buộc"
  value={tags}
  onChange={setTags}
/>
```

**Đúng:**

```tsx
// ✅ Chỉ cần truyền error — color tự động là "error"
<TagsInput
  error
  helpText="Trường này bắt buộc"
  value={tags}
  onChange={setTags}
/>
```

---

### RULE-TAGSINPUT-10: Sử dụng `appendToBody={false}` khi cần suggest list nằm trong container cha

Mặc định `appendToBody` là `true`, suggest list được append vào `document.body`. Khi component nằm trong modal hoặc container có overflow hidden, set `appendToBody={false}` để suggest list nằm trong container cha.

**Sai:**

```tsx
// ❌ Suggest list bị cắt bởi overflow hidden của modal parent
<Modal>
  <TagsInput
    suggestListOptions={options}
    // appendToBody mặc định là true — suggest list append vào body, có thể bị z-index issues
  />
</Modal>
```

**Đúng:**

```tsx
// ✅ Mặc định appendToBody={true} hoạt động tốt trong hầu hết trường hợp
<TagsInput
  suggestListOptions={options}
  renderSuggestItem={(option) => <span>{option}</span>}
/>

// ✅ Dùng appendToBody={false} khi cần suggest list nằm trong container cha
<div className="relative">
  <TagsInput
    appendToBody={false}
    suggestListOptions={options}
    renderSuggestItem={(option) => <span>{option}</span>}
  />
</div>
```
