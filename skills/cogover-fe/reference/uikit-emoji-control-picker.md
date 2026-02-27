## EmojiControlPicker Component (`uikit-emoji-control-picker`)

Đường dẫn: `ui-kit/src/components/EmojiControlPicker`

Bảng chọn emoji với tìm kiếm, phân loại theo danh mục, và lưu 10 emoji đã dùng gần nhất. Kích thước mặc định: `380px × 500px`. Dữ liệu: Twitter emoji từ `@emoji-mart/data/sets/14/twitter.json`.

Props: `onChange` (bắt buộc), `classNames`, `value`, `disabled`.

---

### RULE-EMOJI-PICKER-01: `onChange` nhận `string` emoji native — không phải emoji object

```tsx
// ❌ Không truyền onChange
<EmojiControlPicker />

// ❌ onChange chỉ trả về string, không phải object
<EmojiControlPicker onChange={(emoji) => setEmoji(emoji.native)} />

// ✅
<EmojiControlPicker onChange={(emoji) => setSelectedEmoji(emoji)} />
<EmojiControlPicker onChange={setSelectedEmoji} />
```

---

### RULE-EMOJI-PICKER-02: Dùng `classNames` để tuỳ chỉnh style — không wrap trong div

```tsx
// ❌ Layout lồng nhau, shadow/border bị duplicate
<div className={cx("rounded-lg shadow-lg")}><EmojiControlPicker onChange={handleEmojiSelect} /></div>

// ✅
<EmojiControlPicker classNames="rounded-lg shadow-lg" onChange={handleEmojiSelect} />
```

---

### RULE-EMOJI-PICKER-03: Component tự quản lý frequently used — không tự build bên ngoài

```tsx
// ❌ Tự quản lý danh sách recently used
const [recentEmojis, setRecentEmojis] = useState<string[]>([]);
// ...

// ✅ Component đã tự xử lý internally
<EmojiControlPicker onChange={setSelectedEmoji} />
```

---

### RULE-EMOJI-PICKER-04: Tích hợp sẵn tìm kiếm — không thêm search input bên ngoài

```tsx
// ❌ Thêm input tìm kiếm ngoài — trùng lặp, confuse người dùng
<div>
  <input placeholder="Tìm emoji..." onChange={(e) => setSearch(e.target.value)} />
  <EmojiControlPicker onChange={handleSelect} />
</div>

// ✅
<EmojiControlPicker onChange={handleSelect} />
```

---

### RULE-EMOJI-PICKER-05: Kết hợp với `Popper` để hiển thị dạng dropdown

```tsx
// ❌ Tự quản lý show/hide — không có click-outside, không có positioning
const [show, setShow] = useState(false);
<div className={cx("relative")}>
  <button onClick={() => setShow(!show)}>Emoji</button>
  {show && <div className={cx("absolute top-full left-0 z-50")}><EmojiControlPicker onChange={handleSelect} /></div>}
</div>

// ✅
<Popper
  placement="bottom-start"
  render={({ attributes, ...params }) => (
    <div {...params} {...attributes}><EmojiControlPicker onChange={handleSelect} /></div>
  )}
>
  {({ ref, onClick }) => <button ref={ref} onClick={onClick}>Emoji</button>}
</Popper>
```
