---
title: Cách sử dụng SearchInput Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng SearchInput sai dẫn đến việc thiếu chức năng xoá, quản lý state không đúng và tuỳ chỉnh giao diện sai cách
tags: search, input, search-input, clear, ui-kit
---

## Cách sử dụng SearchInput Component của ui-kit

Đường dẫn component: `ui-kit/src/components/SearchInput`

Props: `value`, `onChange`, `onClear`, `iconClear`, `className`. Hỗ trợ tất cả `React.InputHTMLAttributes` (trừ `value` và `onChange` đã được override) và `ref` forwarding.

---

### RULE-SINPUT-01: Luôn truyền `value` và `onChange` — đây là controlled component

`SearchInput` là controlled component bắt buộc. `value` (kiểu `string`) và `onChange` là required props. Không sử dụng như uncontrolled input.

**Sai:**

```tsx
// ❌ Thiếu value và onChange — component không hoạt động đúng
<SearchInput placeholder="Tìm kiếm..." />
```

**Đúng:**

```tsx
// ✅ Truyền đầy đủ value và onChange
const [search, setSearch] = useState("");

<SearchInput
  value={search}
  onChange={(e) => setSearch(e.target.value)}
  placeholder="Tìm kiếm..."
/>
```

---

### RULE-SINPUT-02: Sử dụng prop `onClear` để hiển thị nút xoá, không tự tạo nút xoá bên ngoài

Khi truyền `onClear`, component tự động hiển thị nút xoá (icon X) khi `value` có giá trị. Không cần tự tạo nút xoá bên ngoài hoặc wrap thêm container.

**Sai:**

```tsx
// ❌ Tự tạo nút xoá bên ngoài — dư thừa và phá vỡ layout
<div className={cx("flex items-center")}>
  <SearchInput value={search} onChange={handleChange} />
  {search && <button onClick={() => setSearch("")}>✕</button>}
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng onClear — nút xoá hiển thị tự động khi có value
<SearchInput
  value={search}
  onChange={(e) => setSearch(e.target.value)}
  onClear={() => setSearch("")}
  placeholder="Tìm kiếm..."
/>
```

---

### RULE-SINPUT-03: Chỉ truyền `iconClear` khi cần thay đổi icon xoá mặc định

`iconClear` mặc định là icon `faXmark` (dấu X). Chỉ truyền khi cần thay đổi sang icon khác. Không truyền lại giá trị mặc định.

**Sai:**

```tsx
// ❌ Truyền lại icon mặc định — dư thừa
<SearchInput
  value={search}
  onChange={handleChange}
  onClear={handleClear}
  iconClear={<FontAwesomeIcon icon={faXmark} fontSize={14} />}
/>
```

**Đúng:**

```tsx
// ✅ Bỏ qua iconClear khi dùng giá trị mặc định
<SearchInput
  value={search}
  onChange={handleChange}
  onClear={handleClear}
/>

// ✅ Chỉ truyền khi cần icon khác
<SearchInput
  value={search}
  onChange={handleChange}
  onClear={handleClear}
  iconClear={<FontAwesomeIcon icon={faCircleXmark} fontSize={14} />}
/>
```

---

### RULE-SINPUT-04: Sử dụng `className` để tuỳ chỉnh kích thước và style container

`className` được áp dụng cho container bên ngoài. Sử dụng nó để thay đổi kích thước (`w-`, `h-`) hoặc thêm margin/padding. Không tạo wrapper div bên ngoài chỉ để điều chỉnh kích thước.

**Sai:**

```tsx
// ❌ Tạo wrapper div chỉ để điều chỉnh kích thước
<div className={cx("w-[18.75rem]")}>
  <SearchInput value={search} onChange={handleChange} />
</div>
```

**Đúng:**

```tsx
// ✅ Truyền trực tiếp qua className
<SearchInput
  className={cx("w-[18.75rem]")}
  value={search}
  onChange={handleChange}
/>
```

---

### RULE-SINPUT-05: Nút xoá chỉ hiển thị khi có cả `onClear` và `value`

Nút xoá chỉ render khi đồng thời có `onClear`, `iconClear` (mặc định đã có) và `value` không rỗng. Nếu chỉ truyền `onClear` mà `value` là chuỗi rỗng thì nút xoá sẽ tự động ẩn, không cần xử lý thêm.

**Sai:**

```tsx
// ❌ Tự ẩn/hiện nút xoá bằng logic bên ngoài — component đã xử lý
<SearchInput
  value={search}
  onChange={handleChange}
  onClear={search ? () => setSearch("") : undefined}
/>
```

**Đúng:**

```tsx
// ✅ Luôn truyền onClear — component tự ẩn khi value rỗng
<SearchInput
  value={search}
  onChange={handleChange}
  onClear={() => setSearch("")}
/>
```

---

## Tham chiếu style mặc định

| Thuộc tính | Giá trị |
|------------|---------|
| Chiều cao | `h-[40px]` |
| Chiều rộng | `w-[190px] max-w-full` |
| Border | `border border-divider-primary` |
| Border radius | `rounded` |
| Màu chữ input | `text-typo-primary` |
| Màu placeholder | `text-typo-description` |
| Typography input | `prose-body2` |
| Icon search | `faSearch` (14px) |
| Icon clear | `faXmark` (14px) |
