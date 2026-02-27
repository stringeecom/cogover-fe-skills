## EmojiPicker Component (`uikit-emoji-picker`)

Đường dẫn: `ui-kit/src/components/EmojiPicker`

Dropdown picker cho phép chọn emoji/icon từ danh sách. Dùng `Popper` internally.

### Props

| Prop | Kiểu | Bắt buộc | Mặc định | Mô tả |
|------|------|----------|----------|-------|
| `icons` | `EmojiListValue[]` | Có | — | Danh sách emoji/icon |
| `value` | `EmojiListValue[]` | Có | — | Danh sách đã chọn |
| `onChange` | `(value: EmojiListValue[]) => void` | Có | — | Callback thay đổi |
| `fullWidth` | `boolean` | Không | `false` | Chiều rộng 100% |
| `singleSelect` | `boolean` | Không | `false` | Chỉ chọn một emoji |
| `disabled` | `boolean` | Không | `false` | Vô hiệu hoá |

```ts
interface EmojiListValue {
  name: string;  // Tên hiển thị (tooltip và key so sánh)
  icon: string;  // Emoji unicode, URL ảnh, hoặc emoji code (CDN)
}
```

---

### RULE-EMOJI-01: Luôn truyền đủ 3 props bắt buộc: `icons`, `value`, `onChange`

```tsx
// ❌ Thiếu value và onChange
<EmojiPicker icons={emojiList} />

// ✅
const [value, setValue] = useState<EmojiListValue[]>([]);
<EmojiPicker icons={emojiList} value={value} onChange={setValue} />
```

---

### RULE-EMOJI-02: Dùng `singleSelect` thay vì tự giới hạn trong `onChange`

```tsx
// ❌
<EmojiPicker icons={emojiList} value={value} onChange={(v) => setValue(v.slice(-1))} />

// ✅
<EmojiPicker singleSelect icons={emojiList} value={value} onChange={setValue} />
```

---

### RULE-EMOJI-03: Dùng `disabled` thay vì tự chặn tương tác bằng style

```tsx
// ❌
<div style={{ pointerEvents: "none", opacity: 0.5 }}>
  <EmojiPicker icons={emojiList} value={value} onChange={setValue} />
</div>

// ✅
<EmojiPicker disabled icons={emojiList} value={value} onChange={setValue} />
```

---

### RULE-EMOJI-04: Cấu trúc `icons` — 3 định dạng icon được hỗ trợ

```tsx
// Emoji unicode trực tiếp
{ icon: "😂", name: "Cười ra nước mắt" }

// URL ảnh custom
{ icon: "https://example.com/icon1.png", name: "Icon 1" }

// Emoji code (CDN twitter)
{ icon: "1f600", name: "Grinning Face" }
```

---

### RULE-EMOJI-05: Component đã có nút xoá tích hợp (hover) — không tự tạo bên ngoài

```tsx
// ✅ Nút xoá hiển thị khi hover, gọi onChange([]) internally
<EmojiPicker icons={emojiList} value={value} onChange={setValue} />

// Xoá từ logic bên ngoài
const handleReset = () => setValue([]);
```
