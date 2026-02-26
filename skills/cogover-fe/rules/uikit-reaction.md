---
title: Cách sử dụng Reaction Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng Reaction sai dẫn đến việc tự render danh sách emoji thủ công, truyền options sai format, và xử lý controlled/uncontrolled state không đúng
tags: reaction, emoji, icon, feedback, ui-kit
---

## Cách sử dụng Reaction Component của ui-kit

Đường dẫn component: `ui-kit/src/components/Reaction`

Props: `value`, `defaultValue`, `onChange`, `options`, `gap`, `direction`, `iconSize`, `disabled`, `tabIndex`. Hỗ trợ tất cả `React.HTMLAttributes<HTMLDivElement>` và `ref` forwarding.

---

### RULE-REACTION-01: Sử dụng options mặc định khi chỉ cần 5 emoji cơ bản (angry, sad, confused, like, love)

Khi không truyền `options` hoặc truyền mảng rỗng, component sẽ tự động sử dụng 5 emoji mặc định: angry, sad, confused, like, love. KHÔNG tự tạo lại danh sách options trùng với mặc định.

**Sai:**

```tsx
// ❌ Tạo lại danh sách options trùng với mặc định
<Reaction
  options={[
    { value: "angry", name: "Angry", icon: "/static/images/icons/reaction/angry.svg" },
    { value: "sad", name: "Sad", icon: "/static/images/icons/reaction/sad.svg" },
    { value: "confused", name: "Confused", icon: "/static/images/icons/reaction/confused.svg" },
    { value: "like", name: "Like", icon: "/static/images/icons/reaction/like.svg" },
    { value: "love", name: "Love", icon: "/static/images/icons/reaction/love.svg" },
  ]}
  onChange={handleChange}
/>
```

**Đúng:**

```tsx
// ✅ Bỏ qua options để dùng mặc định
<Reaction onChange={handleChange} />
```

---

### RULE-REACTION-02: Truyền `options` đúng format `ReactionOption` khi cần tuỳ chỉnh danh sách emoji

Mỗi option phải có đủ 3 trường: `value` (string | number), `name` (string - dùng làm tooltip), `icon` (string - URL ảnh hoặc emoji). KHÔNG thiếu trường `name` vì nó được dùng làm nội dung tooltip.

**Sai:**

```tsx
// ❌ Thiếu trường name — tooltip sẽ không hiển thị đúng
<Reaction
  options={[
    { value: 1, icon: "👍" },
    { value: 2, icon: "👎" },
  ]}
/>
```

**Đúng:**

```tsx
// ✅ Đủ 3 trường cho mỗi option
<Reaction
  options={[
    { value: 1, name: "Thích", icon: "👍" },
    { value: 2, name: "Không thích", icon: "👎" },
    { value: 3, name: "Yêu thích", icon: "❤️" },
  ]}
  onChange={handleChange}
/>
```

---

### RULE-REACTION-03: Sử dụng controlled mode (`value` + `onChange`) hoặc uncontrolled mode (`defaultValue`), không trộn lẫn

Component hỗ trợ cả 2 mode. Khi truyền `value`, component hoạt động ở controlled mode — giá trị hiển thị luôn theo `value` từ bên ngoài. Khi chỉ truyền `defaultValue`, component tự quản lý state nội bộ. KHÔNG truyền cả `value` và `defaultValue` cùng lúc.

**Sai:**

```tsx
// ❌ Trộn lẫn controlled và uncontrolled
<Reaction value={selected} defaultValue="like" onChange={setSelected} />
```

**Đúng:**

```tsx
// ✅ Controlled mode
<Reaction value={selected} onChange={setSelected} />

// ✅ Uncontrolled mode với giá trị khởi tạo
<Reaction defaultValue="like" onChange={handleChange} />

// ✅ Uncontrolled mode không có giá trị khởi tạo
<Reaction onChange={handleChange} />
```

---

### RULE-REACTION-04: Sử dụng prop `direction` để thay đổi hướng hiển thị, không dùng CSS thủ công

Component hỗ trợ `direction` với 2 giá trị: `"horizontal"` (mặc định) và `"vertical"`. KHÔNG tự thêm CSS flex-direction.

**Sai:**

```tsx
// ❌ Override flex-direction thủ công
<Reaction className={cx("flex-col")} onChange={handleChange} />
```

**Đúng:**

```tsx
// ✅ Sử dụng prop direction
<Reaction direction="vertical" onChange={handleChange} />

// ✅ Horizontal là mặc định, không cần truyền
<Reaction onChange={handleChange} />
```

---

### RULE-REACTION-05: Sử dụng prop `gap` và `iconSize` để tuỳ chỉnh khoảng cách và kích thước, không dùng CSS thủ công

`gap` (mặc định: 8, đơn vị px) điều chỉnh khoảng cách giữa các icon. `iconSize` (mặc định: 30, đơn vị px) điều chỉnh kích thước mỗi icon. KHÔNG tự thêm CSS gap hoặc width/height cho các item.

**Sai:**

```tsx
// ❌ Override gap và kích thước bằng CSS thủ công
<Reaction
  className={cx("gap-[1.5rem] [&>div]:w-[2.5rem] [&>div]:h-[2.5rem]")}
  onChange={handleChange}
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng prop gap và iconSize
<Reaction gap={24} iconSize={40} onChange={handleChange} />
```

---

### RULE-REACTION-06: Xử lý giá trị `null` trong callback `onChange` khi người dùng bỏ chọn

Khi người dùng click vào reaction đã chọn, component sẽ gọi `onChange(null)` để bỏ chọn (toggle behavior). Callback `onChange` phải xử lý được giá trị `null`.

**Sai:**

```tsx
// ❌ Không xử lý trường hợp null — có thể gây lỗi runtime
const handleChange = (value: ReactionValue) => {
  setSelected(value);
  submitReaction(value); // Lỗi khi value là null
};
```

**Đúng:**

```tsx
// ✅ Xử lý cả trường hợp chọn và bỏ chọn
const handleChange = (value: ReactionValue | null) => {
  setSelected(value);
  if (value) {
    submitReaction(value);
  } else {
    removeReaction();
  }
};
```

---

## Tham chiếu giá trị mặc định

| Prop | Giá trị mặc định | Mô tả |
|------|-------------------|--------|
| `gap` | `8` (px) | Khoảng cách giữa các icon |
| `iconSize` | `30` (px) | Kích thước mỗi icon (width & height) |
| `direction` | `"horizontal"` | Hướng hiển thị |
| `tabIndex` | `0` | Tab index cho accessibility |
| `disabled` | `undefined` | Trạng thái vô hiệu hoá |

## Danh sách emoji mặc định

| Value | Name | Mô tả |
|-------|------|--------|
| `"angry"` | Angry | Biểu cảm tức giận |
| `"sad"` | Sad | Biểu cảm buồn |
| `"confused"` | Confused | Biểu cảm bối rối |
| `"like"` | Like | Biểu cảm thích |
| `"love"` | Love | Biểu cảm yêu thích |
