# Quy tắc EmojiPicker Component (`uikit-emoji-picker`)

Component `EmojiPicker` từ `ui-kit/src/components/EmojiPicker` là dropdown picker cho phép người dùng chọn emoji hoặc icon từ danh sách. Sử dụng `Popper` internally để hiển thị popup chứa danh sách emoji có hỗ trợ tìm kiếm.

---

## Props

| Prop | Kiểu | Bắt buộc | Mặc định | Mô tả |
|------|------|----------|----------|-------|
| `icons` | `EmojiListValue[]` | Có | — | Danh sách emoji/icon để hiển thị trong dropdown |
| `value` | `EmojiListValue[]` | Có | — | Danh sách emoji đã được chọn |
| `onChange` | `(value: EmojiListValue[]) => void` | Có | — | Callback khi giá trị thay đổi |
| `fullWidth` | `boolean` | Không | `false` | Hiển thị picker với chiều rộng 100% |
| `singleSelect` | `boolean` | Không | `false` | Chỉ cho phép chọn một emoji duy nhất |
| `disabled` | `boolean` | Không | `false` | Vô hiệu hoá picker |

Ngoài ra, `EmojiPicker` kế thừa tất cả `React.HTMLAttributes<HTMLDivElement>` (ngoại trừ `onChange`).

### Kiểu dữ liệu `EmojiListValue`

```ts
interface EmojiListValue {
    name: string;  // Tên hiển thị (dùng làm tooltip và key để so sánh)
    icon: string;  // Emoji unicode, URL ảnh (https://...), hoặc emoji code (render qua CDN)
}
```

---

## RULE-EMOJI-01 -- Luôn truyền đủ 3 props bắt buộc: `icons`, `value`, `onChange`

`EmojiPicker` là controlled component, bắt buộc phải truyền `icons` (danh sách emoji), `value` (giá trị đã chọn), và `onChange` (callback xử lý thay đổi).

**Sai:**
```tsx
// ❌ Thiếu value và onChange -- component sẽ không hoạt động đúng
<EmojiPicker icons={emojiList} />
```

**Đúng:**
```tsx
// ✅ Truyền đủ 3 props bắt buộc
const [value, setValue] = useState<EmojiListValue[]>([]);

<EmojiPicker icons={emojiList} value={value} onChange={setValue} />
```

---

## RULE-EMOJI-02 -- Sử dụng `singleSelect` khi chỉ cho phép chọn một emoji

Khi logic chỉ cần chọn duy nhất một emoji, truyền `singleSelect` thay vì tự xử lý logic giới hạn trong `onChange`.

**Sai:**
```tsx
// ❌ Tự giới hạn số lượng trong onChange
<EmojiPicker
    icons={emojiList}
    value={value}
    onChange={(newValue) => setValue(newValue.slice(-1))}
/>
```

**Đúng:**
```tsx
// ✅ Sử dụng singleSelect -- component tự xử lý logic toggle chọn/bỏ chọn
<EmojiPicker
    singleSelect
    icons={emojiList}
    value={value}
    onChange={setValue}
/>
```

---

## RULE-EMOJI-03 -- Sử dụng `fullWidth` thay vì wrapper hoặc style thủ công

Khi cần picker chiếm toàn bộ chiều rộng container, sử dụng prop `fullWidth` thay vì tự style.

**Sai:**
```tsx
// ❌ Wrap thủ công để fullWidth
<div style={{ width: "100%" }}>
    <EmojiPicker icons={emojiList} value={value} onChange={setValue} />
</div>
```

**Đúng:**
```tsx
// ✅ Sử dụng prop fullWidth
<EmojiPicker fullWidth icons={emojiList} value={value} onChange={setValue} />
```

---

## RULE-EMOJI-04 -- Sử dụng `disabled` thay vì tự chặn tương tác

Khi cần vô hiệu hoá picker (ví dụ: trạng thái chỉ đọc), sử dụng prop `disabled`. Prop này sẽ vô hiệu hoá cả Popper (không mở dropdown) và ẩn nút xoá.

**Sai:**
```tsx
// ❌ Tự chặn tương tác bằng style
<div style={{ pointerEvents: "none", opacity: 0.5 }}>
    <EmojiPicker icons={emojiList} value={value} onChange={setValue} />
</div>
```

**Đúng:**
```tsx
// ✅ Sử dụng prop disabled -- tự động vô hiệu hoá Popper và ẩn nút xoá
<EmojiPicker disabled icons={emojiList} value={value} onChange={setValue} />
```

---

## RULE-EMOJI-05 -- Cấu trúc đúng cho `icons` với các loại icon khác nhau

Trường `icon` trong `EmojiListValue` hỗ trợ 3 định dạng: emoji unicode trực tiếp, URL ảnh (bắt đầu bằng `https` hoặc `/static/`), và emoji code (render qua CDN twitter). Đảm bảo truyền đúng định dạng.

```tsx
// ✅ Emoji unicode trực tiếp
const unicodeIcons: EmojiListValue[] = [
    { icon: "😂", name: "Cười ra nước mắt" },
    { icon: "❤️", name: "Tim đỏ" },
];

// ✅ URL ảnh custom
const imageIcons: EmojiListValue[] = [
    { icon: "https://example.com/icon1.png", name: "Icon 1" },
    { icon: "/static/icons/star.png", name: "Ngôi sao" },
];

// ✅ Emoji code (render qua CDN twitter)
const codeIcons: EmojiListValue[] = [
    { icon: "1f600", name: "Grinning Face" },
];
```

---

## RULE-EMOJI-06 -- Xoá tất cả giá trị bằng cách gọi `onChange([])`, không tự reset state

Component đã có sẵn nút xoá (icon X) hiển thị khi hover hoặc khi dropdown đang mở. Nút này gọi `onChange([])` internally. Khi cần xoá giá trị từ bên ngoài, cũng sử dụng `onChange([])`.

**Sai:**
```tsx
// ❌ Tự tạo nút xoá bên ngoài
<div>
    <EmojiPicker icons={emojiList} value={value} onChange={setValue} />
    <button onClick={() => setValue([])}>Xoá</button>
</div>
```

**Đúng:**
```tsx
// ✅ Component đã có nút xoá tích hợp, hiển thị khi hover
<EmojiPicker icons={emojiList} value={value} onChange={setValue} />

// Nếu cần xoá từ logic bên ngoài, gọi trực tiếp setState
const handleReset = () => setValue([]);
```

---

## Ví dụ sử dụng đầy đủ

### Chọn nhiều emoji (mặc định)

```tsx
import { useState } from "react";
import EmojiPicker from "ui-kit/src/components/EmojiPicker";
import { EmojiListValue } from "ui-kit/src/components/EmojiPicker/EmojiList/type";

const emojiList: EmojiListValue[] = [
    { icon: "😂", name: "Cười" },
    { icon: "😍", name: "Thích" },
    { icon: "👍", name: "Tốt" },
    { icon: "❤️", name: "Yêu" },
];

function MultiSelectExample() {
    const [value, setValue] = useState<EmojiListValue[]>([]);

    return (
        <EmojiPicker
            fullWidth
            icons={emojiList}
            value={value}
            onChange={setValue}
        />
    );
}
```

### Chọn một emoji duy nhất

```tsx
function SingleSelectExample() {
    const [value, setValue] = useState<EmojiListValue[]>([]);

    return (
        <EmojiPicker
            singleSelect
            fullWidth
            icons={emojiList}
            value={value}
            onChange={setValue}
        />
    );
}
```
