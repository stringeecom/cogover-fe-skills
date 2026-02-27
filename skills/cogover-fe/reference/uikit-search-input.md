## SearchInput Component (`uikit-search-input`)

Đường dẫn: `ui-kit/src/components/SearchInput`

Props: `value` (required), `onChange` (required), `onClear`, `iconClear`, `className`. Hỗ trợ `React.InputHTMLAttributes` và `ref` forwarding.

Style mặc định: `h-[40px]`, `w-[190px] max-w-full`, `border border-divider-primary`, `rounded`.

---

### RULE-SINPUT-01: `SearchInput` là controlled component — luôn truyền `value` và `onChange`

```tsx
// ❌ Thiếu value và onChange
<SearchInput placeholder="Tìm kiếm..." />

// ✅
const [search, setSearch] = useState("");
<SearchInput value={search} onChange={(e) => setSearch(e.target.value)} placeholder="Tìm kiếm..." />
```

---

### RULE-SINPUT-02: Dùng prop `onClear` để hiển thị nút xoá tự động

Nút xoá chỉ hiển thị khi có `onClear` VÀ `value` không rỗng.

```tsx
// ❌ Tự tạo nút xoá bên ngoài — dư thừa, phá vỡ layout
<div className={cx("flex items-center")}>
  <SearchInput value={search} onChange={handleChange} />
  {search && <button onClick={() => setSearch("")}>✕</button>}
</div>

// ✅ Nút xoá hiển thị tự động khi có value
<SearchInput
  value={search}
  onChange={(e) => setSearch(e.target.value)}
  onClear={() => setSearch("")}
  placeholder="Tìm kiếm..."
/>
```

---

### RULE-SINPUT-03: Luôn truyền `onClear` — component tự ẩn khi value rỗng

```tsx
// ❌ Tự quản lý logic ẩn/hiện onClear
<SearchInput value={search} onChange={handleChange} onClear={search ? () => setSearch("") : undefined} />

// ✅
<SearchInput value={search} onChange={handleChange} onClear={() => setSearch("")} />
```

---

### RULE-SINPUT-04: Dùng `className` để tuỳ chỉnh container — không wrap thêm div

```tsx
// ❌
<div className={cx("w-[18.75rem]")}><SearchInput value={search} onChange={handleChange} /></div>

// ✅
<SearchInput className={cx("w-[18.75rem]")} value={search} onChange={handleChange} />
```
