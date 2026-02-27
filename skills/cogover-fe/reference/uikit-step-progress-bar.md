---
title: Cách sử dụng component StepProgressBar trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến hành vi màu sắc sai, các phần không hiển thị hoặc render thông tin bottom bị lỗi
tags: progress, step, bar, ui-kit
---

## Cách sử dụng component StepProgressBar trong ui-kit

Đường dẫn component: `ui-kit/src/components/StepProgressBar`

Props: `percentage` (bắt buộc), `stepNumber`, `title`, `subtitle`, `color`, `backgroundColor`, `stepBackgroundColor`, `stepTextColor`, `showStepNumber`, `showTitle`, `showSubtitle`, `showPercentage`, `showProgressBar`, `showBottomInfo`, `progressHeight`, `progressBorderRadius`, `stepSize`, `stepFontSize`, `percentageColor`, `percentageFontSize`, `rightElement`, `bottomInfo`, `bottomInfoIcon`, `bottomInfoText`, `className`, `titleClassName`, `subtitleClassName`, `percentageClassName`, `bottomInfoClassName`, `progressClassName`.

---

### RULE-SPB-01: `percentage` là prop duy nhất bắt buộc — luôn truyền giá trị hợp lệ từ 0 đến 100

Component nội bộ clamp giá trị về `[0, 100]`, nhưng bạn không nên dựa vào điều này. Luôn đảm bảo giá trị hợp lệ trước khi truyền vào.

**Sai:**

```tsx
// ❌ Thiếu percentage — lỗi TypeScript
<StepProgressBar title="Step 1" />

// ❌ Giá trị có thể là NaN hoặc Infinity khi totalCount là 0
<StepProgressBar percentage={completedCount / totalCount * 100} />
```

**Đúng:**

```tsx
// ✅ Luôn bảo vệ chống lại các giá trị không hợp lệ
const percentage = totalCount > 0 ? (completedCount / totalCount) * 100 : 0;

<StepProgressBar percentage={percentage} />;
```

---

### RULE-SPB-02: Hiểu hành vi màu vòng tròn step trước khi tùy chỉnh

Màu vòng tròn phụ thuộc vào việc `stepBackgroundColor` có được cung cấp hay không:

- **`stepBackgroundColor` không được cung cấp**: background vòng tròn được tự động làm sáng từ `color` (trộn 85% trắng); text vòng tròn sử dụng chính `color`.
- **`stepBackgroundColor` được cung cấp**: background vòng tròn sử dụng giá trị đó; text vòng tròn sử dụng `stepTextColor` (mặc định `#ffffff`).

`stepTextColor` **không có hiệu lực** trừ khi `stepBackgroundColor` cũng được cung cấp.

**Sai:**

```tsx
// ❌ stepTextColor bị bỏ qua — stepBackgroundColor là bắt buộc để nó có hiệu lực
<StepProgressBar percentage={50} stepNumber={1} stepTextColor="#000000" />
```

**Đúng:**

```tsx
// ✅ Hành vi mặc định — background vòng tròn tự động làm sáng từ color, text sử dụng color
<StepProgressBar percentage={50} stepNumber={1} color="#6366f1" />

// ✅ Tùy chỉnh đầy đủ — phải cung cấp cả stepBackgroundColor và stepTextColor
<StepProgressBar
  percentage={50}
  stepNumber={1}
  color="#6366f1"
  stepBackgroundColor="#6366f1"
  stepTextColor="#ffffff"
/>
```

---

### RULE-SPB-03: Vòng tròn step chỉ render khi cả `showStepNumber=true` VÀ `stepNumber` được cung cấp

`showStepNumber` mặc định là `true`, nhưng vòng tròn sẽ không render nếu `stepNumber` là `undefined`. Cả hai điều kiện phải được đáp ứng.

**Sai:**

```tsx
// ❌ Vòng tròn không render — stepNumber không được cung cấp
<StepProgressBar percentage={50} showStepNumber={true} />

// ❌ Vòng tròn không render — showStepNumber là false
<StepProgressBar percentage={50} stepNumber={1} showStepNumber={false} />
```

**Đúng:**

```tsx
// ✅ Truyền stepNumber và để showStepNumber ở giá trị mặc định (true)
<StepProgressBar percentage={50} stepNumber={1} />

// ✅ Để ẩn vòng tròn, đơn giản bỏ qua stepNumber — không cần set showStepNumber={false}
<StepProgressBar percentage={50} title="Progress" />
```

---

### RULE-SPB-04: Thông tin bottom tự ẩn khi không có nội dung — không set `showBottomInfo={false}` không cần thiết

Component kiểm tra `(bottomInfo || bottomInfoText)` trước khi render phần bottom. Nếu không có cái nào được cung cấp, phần này tự động ẩn ngay cả khi `showBottomInfo=true`. Chỉ sử dụng `showBottomInfo={false}` khi bạn cần force-hide nó mặc dù có dữ liệu.

**Sai:**

```tsx
// ❌ Không cần thiết — phần bottom đã tự ẩn khi không có nội dung được cung cấp
<StepProgressBar percentage={50} showBottomInfo={false} />
```

**Đúng:**

```tsx
// ✅ Sử dụng bottomInfoText cho nội dung text đơn giản
<StepProgressBar
  percentage={60}
  bottomInfoText="6 of 10 tasks completed"
/>

// ✅ Sử dụng bottomInfo cho ReactNode tùy ý
<StepProgressBar
  percentage={60}
  bottomInfo={<span>Overdue!</span>}
/>

// ✅ Sử dụng showBottomInfo={false} chỉ để force-hide mặc dù có nội dung
<StepProgressBar
  percentage={60}
  bottomInfoText="6 of 10 tasks completed"
  showBottomInfo={false}
/>
```

---

### RULE-SPB-05: `bottomInfo` ưu tiên hơn `bottomInfoText` — không truyền cả hai cùng lúc

Khi cả `bottomInfo` và `bottomInfoText` đều được cung cấp, `bottomInfo` (ReactNode) được render và `bottomInfoText` bị bỏ qua âm thầm.

**Sai:**

```tsx
// ❌ bottomInfoText bị bỏ qua hoàn toàn khi bottomInfo có mặt
<StepProgressBar
  percentage={50}
  bottomInfo={<CustomInfo />}
  bottomInfoText="6 of 10 tasks completed"
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng một trong hai, không bao giờ cả hai
<StepProgressBar percentage={50} bottomInfoText="6 of 10 tasks completed" />

<StepProgressBar percentage={50} bottomInfo={<CustomInfo />} />
```

---

### RULE-SPB-06: Sử dụng prop `color` làm single source of truth — không override màu qua wrapper styles

`color` kiểm soát progress bar fill, màu text percentage và màu vòng tròn step cùng lúc. Sử dụng `percentageColor` để override màu percentage độc lập nếu cần.

**Sai:**

```tsx
// ❌ Override màu bên ngoài component khiến bar và percentage không đồng bộ
<div style={{ color: "#6366f1" }}>
  <StepProgressBar percentage={50} />
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng prop color — bar, vòng tròn và percentage giữ đồng bộ
<StepProgressBar percentage={50} color="#6366f1" />

// ✅ Override màu percentage độc lập khi cần
<StepProgressBar percentage={50} color="#6366f1" percentageColor="#000000" />
```

---

### RULE-SPB-07: Sử dụng `rightElement` cho avatars hoặc badges bên cạnh title — không tự build wrapper layout

Component đã có layout flex sẵn cho `rightElement`, được đặt giữa block title và percentage. Không tạo wrapper bên ngoài để đạt được điều này.

**Sai:**

```tsx
// ❌ Wrapper tùy chỉnh phá vỡ alignment với hiển thị percentage
<div className="flex items-center gap-2">
  <StepProgressBar percentage={70} title="Team A" />
  <Avatar src={avatarUrl} size={24} />
</div>
```

**Đúng:**

```tsx
// ✅ rightElement được đặt đúng giữa title và percentage
<StepProgressBar
  percentage={70}
  title="Team A"
  rightElement={<Avatar src={avatarUrl} size={24} />}
/>
```
