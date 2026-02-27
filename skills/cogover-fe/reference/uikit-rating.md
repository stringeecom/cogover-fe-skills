## Rating Component (`uikit-rating`)

Đường dẫn: `ui-kit/src/components/Rating`

Props: `value`, `defaultValue`, `onChange`, `max` (mặc định 5), `icon`, `iconSize` (mặc định 30px), `gap` (mặc định 8px), `direction` ("horizontal"|"vertical"), `disabled`, `tabIndex` (+ `React.HTMLAttributes<HTMLDivElement>`).

---

### RULE-RATING-01: Pair `value` với `onChange` trong controlled mode

```tsx
// ❌ Rating bị đóng băng — không thể thay đổi
const [score, setScore] = useState(3);
<Rating value={score} />

// ✅
<Rating value={score} onChange={(val) => setScore(val)} />
```

---

### RULE-RATING-02: Omit `value` cho uncontrolled — dùng `defaultValue` để set giá trị khởi tạo

```tsx
// ❌ Controlled value nhưng không có onChange
<Rating value={3} />

// ✅
<Rating defaultValue={3} />
<Rating />  // Bắt đầu từ null
```

---

### RULE-RATING-03: `onChange` trả về `number | null`, không phải event

```tsx
// ❌
<Rating value={score} onChange={(e) => setScore(Number(e.target.value))} />

// ✅ onChange nhận trực tiếp number | null (0 khi bỏ chọn)
<Rating value={score} onChange={(val) => setScore(val)} />
```

---

### RULE-RATING-04: Dùng prop `icon` để tùy chỉnh — không truyền qua children

```tsx
// ❌ Children không hiển thị icon custom
<Rating><HeartIcon /></Rating>

// ✅ Mặc định (sao), emoji, hoặc URL
<Rating value={score} onChange={setScore} />
<Rating icon="❤️" value={score} onChange={setScore} />
<Rating icon="/static/images/heart.svg" value={score} onChange={setScore} />
```

---

### RULE-RATING-05: Dùng `max` để thay đổi số icon — không render thủ công

```tsx
// ❌ Bỏ qua logic hover và select có sẵn
{Array(10).fill(0).map((_, i) => <Rating key={i} value={i < score ? 1 : 0} max={1} />)}

// ✅
<Rating max={10} value={score} onChange={setScore} />
```

---

### RULE-RATING-06: Dùng `iconSize`, `gap`, `direction` — không override bằng className

```tsx
// ❌ className không override inline style của icon
<Rating className="[&>div]:w-[2.5rem] [&>div]:h-[2.5rem] flex-col" value={score} onChange={setScore} />

// ✅
<Rating iconSize={40} gap={12} direction="vertical" value={score} onChange={setScore} />
```

---

### RULE-RATING-07: Dùng prop `disabled` — không dùng pointer-events-none thủ công

```tsx
// ❌
<div className="pointer-events-none opacity-50"><Rating value={score} onChange={setScore} /></div>

// ✅
<Rating disabled value={score} onChange={setScore} />
```
