---
title: Cách sử dụng component EmojiControlPicker trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến callback onChange không nhận đúng emoji, danh sách frequently used không hoạt động, và tìm kiếm emoji bị lỗi
tags: emoji, emoji-picker, icon, reaction, ui-kit
---

## Cách sử dụng component EmojiControlPicker trong ui-kit

Đường dẫn component: `ui-kit/src/components/EmojiControlPicker`

Props bắt buộc: `onChange`.
Props tùy chọn: `classNames`, `value`, `disabled`.

Component hiển thị bảng chọn emoji với các tính năng: phân loại emoji theo danh mục (Frequently Used, People, Nature, Foods, Activity, Places, Objects, Symbols, Flags, Custom Icon), tìm kiếm emoji, và lưu danh sách emoji đã dùng gần đây (tối đa 10 emoji).

Dữ liệu emoji sử dụng bộ Twitter emoji từ `@emoji-mart/data/sets/14/twitter.json`.

Kích thước mặc định: `380px` x `500px`.

---

### RULE-EMOJI-PICKER-01: Luôn truyền `onChange` callback — đây là prop bắt buộc duy nhất

`onChange` nhận tham số là chuỗi emoji native (ví dụ: "😀", "🎉"). Không truyền handler xử lý emoji object — component đã extract `skins[0].native` internally.

**Sai:**

```tsx
// ❌ Không truyền onChange — component không hoạt động
<EmojiControlPicker />

// ❌ Expect emoji object — onChange chỉ trả về string native
<EmojiControlPicker onChange={(emoji) => setEmoji(emoji.native)} />
```

**Đúng:**

```tsx
// ✅ onChange nhận trực tiếp chuỗi emoji native
<EmojiControlPicker onChange={(emoji) => setSelectedEmoji(emoji)} />

// ✅ Truyền trực tiếp setter
<EmojiControlPicker onChange={setSelectedEmoji} />
```

---

### RULE-EMOJI-PICKER-02: Sử dụng `classNames` để tuỳ chỉnh style container — không wrap trong div có style

`classNames` được merge vào root container thông qua hàm `cx`. Không wrap component trong div để thêm style — sẽ gây layout lồng nhau không cần thiết.

**Sai:**

```tsx
// ❌ Wrap trong div để thêm style — layout bị lồng, shadow/border bị duplicate
<div className={cx("rounded-lg shadow-lg")}>
  <EmojiControlPicker onChange={handleEmojiSelect} />
</div>
```

**Đúng:**

```tsx
// ✅ Dùng classNames để tuỳ chỉnh style trực tiếp
<EmojiControlPicker
  classNames="rounded-lg shadow-lg"
  onChange={handleEmojiSelect}
/>
```

---

### RULE-EMOJI-PICKER-03: Component tự quản lý danh sách frequently used — không tự build bên ngoài

Component tự động lưu và hiển thị tối đa 10 emoji đã chọn gần nhất trong danh mục "frequentlyUsed". Không tự quản lý danh sách frequently used bên ngoài rồi truyền vào.

**Sai:**

```tsx
// ❌ Tự quản lý danh sách frequently used bên ngoài
const [recentEmojis, setRecentEmojis] = useState<string[]>([]);

const handleSelect = (emoji: string) => {
  setRecentEmojis((prev) => [...prev.slice(-9), emoji]);
  setSelectedEmoji(emoji);
};

<div>
  <div>
    {recentEmojis.map((e) => (
      <span key={e}>{e}</span>
    ))}
  </div>
  <EmojiControlPicker onChange={handleSelect} />
</div>
```

**Đúng:**

```tsx
// ✅ Component đã tự quản lý frequently used internally
<EmojiControlPicker onChange={setSelectedEmoji} />
```

---

### RULE-EMOJI-PICKER-04: Component đã tích hợp sẵn tìm kiếm — không tự build search bên ngoài

Component bao gồm `TextField` tìm kiếm emoji theo `id` và `name`. Không thêm input tìm kiếm bên ngoài — sẽ gây trùng lặp và confuse người dùng.

**Sai:**

```tsx
// ❌ Thêm input tìm kiếm bên ngoài — component đã có sẵn
<div>
  <input
    placeholder="Tìm emoji..."
    onChange={(e) => setSearch(e.target.value)}
  />
  <EmojiControlPicker onChange={handleSelect} />
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng trực tiếp — tìm kiếm đã tích hợp sẵn bên trong
<EmojiControlPicker onChange={handleSelect} />
```

---

### RULE-EMOJI-PICKER-05: Kết hợp với Popper để hiển thị dạng dropdown

EmojiControlPicker là một panel tĩnh (380px x 500px). Khi cần hiển thị dưới dạng dropdown/popup, kết hợp với `Popper` component. Không tự quản lý show/hide bằng state + absolute positioning.

**Sai:**

```tsx
// ❌ Tự quản lý show/hide — không có click-outside, không có positioning
const [show, setShow] = useState(false);

<div className={cx("relative")}>
  <button onClick={() => setShow(!show)}>Emoji</button>
  {show && (
    <div className={cx("absolute top-full left-0 z-50")}>
      <EmojiControlPicker onChange={handleSelect} />
    </div>
  )}
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng Popper để hiển thị dạng dropdown
<Popper
  placement="bottom-start"
  render={({ attributes, ...params }) => (
    <div {...params} {...attributes}>
      <EmojiControlPicker onChange={handleSelect} />
    </div>
  )}
>
  {({ ref, onClick }) => (
    <button ref={ref} onClick={onClick}>
      Emoji
    </button>
  )}
</Popper>
```
