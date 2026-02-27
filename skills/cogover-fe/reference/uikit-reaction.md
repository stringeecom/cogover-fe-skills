## Reaction Component (`uikit-reaction`)

Đường dẫn: `ui-kit/src/components/Reaction`

Props: `value`, `defaultValue`, `onChange`, `options`, `gap` (mặc định 8px), `direction` (mặc định "horizontal"), `iconSize` (mặc định 30px), `disabled`, `tabIndex`. Hỗ trợ `React.HTMLAttributes<HTMLDivElement>` và `ref` forwarding.

Emoji mặc định (khi không truyền `options`): angry, sad, confused, like, love.

---

### RULE-REACTION-01: Bỏ qua `options` để dùng 5 emoji mặc định

```tsx
// ❌ Tạo lại danh sách trùng với mặc định — dư thừa
<Reaction options={[
  { value: "angry", name: "Angry", icon: "/static/.../angry.svg" },
  ...
]} onChange={handleChange} />

// ✅
<Reaction onChange={handleChange} />
```

---

### RULE-REACTION-02: Mỗi option cần đủ 3 trường: `value`, `name`, `icon`

```tsx
// ❌ Thiếu name — tooltip không hiển thị
<Reaction options={[{ value: 1, icon: "👍" }]} />

// ✅
<Reaction
  options={[
    { value: 1, name: "Thích", icon: "👍" },
    { value: 2, name: "Không thích", icon: "👎" },
  ]}
  onChange={handleChange}
/>
```

---

### RULE-REACTION-03: Dùng controlled (`value` + `onChange`) hoặc uncontrolled (`defaultValue`), không trộn lẫn

```tsx
// ❌
<Reaction value={selected} defaultValue="like" onChange={setSelected} />

// ✅ Controlled
<Reaction value={selected} onChange={setSelected} />

// ✅ Uncontrolled
<Reaction defaultValue="like" onChange={handleChange} />
```

---

### RULE-REACTION-04: Dùng props `direction`, `gap`, `iconSize` — không override bằng CSS

```tsx
// ❌
<Reaction className={cx("flex-col gap-[1.5rem]")} onChange={handleChange} />

// ✅
<Reaction direction="vertical" gap={24} iconSize={40} onChange={handleChange} />
```

---

### RULE-REACTION-05: Xử lý giá trị `null` trong `onChange` (khi người dùng bỏ chọn)

```tsx
// ❌ Không xử lý null — lỗi runtime khi bỏ chọn
const handleChange = (value: ReactionValue) => submitReaction(value);

// ✅
const handleChange = (value: ReactionValue | null) => {
  if (value) submitReaction(value);
  else removeReaction();
};
```
