## StepProgressBar Component (`uikit-step-progress-bar`)

Đường dẫn: `ui-kit/src/components/StepProgressBar`

Props: `percentage` (bắt buộc), `stepNumber`, `title`, `subtitle`, `color`, `backgroundColor`, `stepBackgroundColor`, `stepTextColor`, `showStepNumber`, `showTitle`, `showSubtitle`, `showPercentage`, `showProgressBar`, `showBottomInfo`, `progressHeight`, `progressBorderRadius`, `stepSize`, `stepFontSize`, `percentageColor`, `percentageFontSize`, `rightElement`, `bottomInfo`, `bottomInfoIcon`, `bottomInfoText`, `className`, `titleClassName`, `subtitleClassName`, `percentageClassName`, `bottomInfoClassName`, `progressClassName`.

---

### RULE-SPB-01: `percentage` là prop duy nhất bắt buộc — truyền giá trị hợp lệ 0–100

```tsx
// ❌ Có thể NaN khi totalCount = 0
<StepProgressBar percentage={completedCount / totalCount * 100} />

// ✅
const percentage = totalCount > 0 ? (completedCount / totalCount) * 100 : 0;
<StepProgressBar percentage={percentage} />
```

---

### RULE-SPB-02: Vòng tròn step chỉ render khi `showStepNumber=true` VÀ `stepNumber` được cung cấp

```tsx
// ❌ Không render — thiếu stepNumber
<StepProgressBar percentage={50} showStepNumber={true} />

// ✅
<StepProgressBar percentage={50} stepNumber={1} />
```

---

### RULE-SPB-03: `stepTextColor` chỉ có hiệu lực khi `stepBackgroundColor` được cung cấp

- Không có `stepBackgroundColor`: background vòng tròn tự động làm sáng từ `color`; text dùng `color`.
- Có `stepBackgroundColor`: background dùng giá trị đó; text dùng `stepTextColor` (mặc định `#ffffff`).

```tsx
// ❌ stepTextColor bị bỏ qua
<StepProgressBar percentage={50} stepNumber={1} stepTextColor="#000000" />

// ✅ Tùy chỉnh đầy đủ
<StepProgressBar percentage={50} stepNumber={1} color="#6366f1" stepBackgroundColor="#6366f1" stepTextColor="#ffffff" />
```

---

### RULE-SPB-04: `bottomInfo` ưu tiên hơn `bottomInfoText` — không dùng cả hai cùng lúc

```tsx
// ❌ bottomInfoText bị bỏ qua khi có bottomInfo
<StepProgressBar percentage={50} bottomInfo={<CustomInfo />} bottomInfoText="6 of 10 tasks" />

// ✅
<StepProgressBar percentage={50} bottomInfoText="6 of 10 tasks completed" />
<StepProgressBar percentage={50} bottomInfo={<CustomInfo />} />
```

---

### RULE-SPB-05: Dùng `color` làm source of truth — không override màu qua wrapper styles

```tsx
// ❌
<div style={{ color: "#6366f1" }}><StepProgressBar percentage={50} /></div>

// ✅
<StepProgressBar percentage={50} color="#6366f1" />
<StepProgressBar percentage={50} color="#6366f1" percentageColor="#000000" />  // Override percentage riêng
```

---

### RULE-SPB-06: Dùng `rightElement` cho avatars/badges cạnh title — không tự build wrapper

```tsx
// ❌
<div className="flex items-center gap-2">
  <StepProgressBar percentage={70} title="Team A" />
  <Avatar src={avatarUrl} size={24} />
</div>

// ✅
<StepProgressBar percentage={70} title="Team A" rightElement={<Avatar src={avatarUrl} size={24} />} />
```
