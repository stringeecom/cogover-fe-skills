---
title: Cách sử dụng component VerticalSteps trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến highlighting step sai, navigation không hoạt động hoặc layout tràn trong các containers bị giới hạn
tags: steps, wizard, vertical, sidebar, ui-kit
---

## Cách sử dụng component VerticalSteps trong ui-kit

Đường dẫn component: `ui-kit/src/components/VerticalSteps`

Props: `currentStep` (bắt buộc, 0-based), `stepLabels` (bắt buộc), `onClickStep`, `isCanClick`, `className`, `classNameIcon`.

---

### RULE-VSTEPS-01: `currentStep` là 0-based — không bao giờ truyền giá trị 1-based

Highlighting step active/completed được xác định bằng cách so sánh `index` với `currentStep`. Truyền `1` khi step đầu tiên đang active khiến step thứ hai xuất hiện active.

**Sai:**

```tsx
// ❌ 1-based — step thứ hai xuất hiện active thay vì step đầu tiên
<VerticalSteps currentStep={1} stepLabels={["Setup", "Config", "Review"]} />
```

**Đúng:**

```tsx
// ✅ 0-based — step đầu tiên được highlight đúng
<VerticalSteps currentStep={0} stepLabels={["Setup", "Config", "Review"]} />
```

---

### RULE-VSTEPS-02: Hiểu ba trạng thái trực quan — past, current và upcoming

Styling step dựa trên quan hệ giữa `index` và `currentStep`:

| Điều kiện               | Vòng tròn | Label   | Đường kết nối |
| ----------------------- | --------- | ------- | ------------- |
| `index < currentStep`   | Primary   | Primary | Primary       |
| `index === currentStep` | Primary   | Primary | Gray          |
| `index > currentStep`   | Gray      | Gray    | Gray          |

Không có **icon "completed" riêng** — các steps completed và current trông giống nhau (màu primary). Sự khác biệt duy nhất là đường kết nối sau step hiện tại.

**Mô hình tinh thần đúng:**

```
currentStep = 1:
  Step 1 (index 0) → vòng tròn primary, đường bên dưới primary
  Step 2 (index 1) → vòng tròn primary, đường bên dưới gray  ← current
  Step 3 (index 2) → vòng tròn gray, đường bên dưới gray
```

---

### RULE-VSTEPS-03: Luôn set `isCanClick` khi cung cấp `onClickStep` — bỏ qua cả hai khi navigation bị vô hiệu hóa

`isCanClick` thêm `cursor-pointer` vào mỗi hàng step. Không có nó, các steps xuất hiện không thể click ngay cả khi `onClickStep` được cung cấp. Ngược lại, không set `isCanClick` mà không có `onClickStep` — nó hiển thị con trỏ pointer không có hành động.

**Sai:**

```tsx
// ❌ onClickStep mà không có isCanClick — không có cursor pointer, UX khó hiểu
<VerticalSteps currentStep={step} stepLabels={steps} onClickStep={setStep} />

// ❌ isCanClick mà không có onClickStep — cursor pointer không có hành động
<VerticalSteps currentStep={step} stepLabels={steps} isCanClick />
```

**Đúng:**

```tsx
// ✅ Cả hai cùng nhau cho các steps có thể navigate
<VerticalSteps
  currentStep={step}
  stepLabels={steps}
  isCanClick
  onClickStep={(index) => setStep(index)}
/>

// ✅ Không có cái nào cho các steps không thể navigate (chỉ hiển thị)
<VerticalSteps currentStep={step} stepLabels={steps} />
```

---

### RULE-VSTEPS-04: `VerticalSteps` có dimensions cố định — sử dụng `className` để override cho các layouts không chuẩn

Component có sizing sẵn (`min-w-[260px] max-w-[260px]`, tablet: `220px`, height `710px`) dự định cho một **sidebar step navigator**. Nếu bạn cần dimensions khác, override qua `className`.

**Sai:**

```tsx
// ❌ Wrap với constraint độ rộng — wrapper bên ngoài đánh nhau với min/max-w nội bộ
<div className="w-40">
  <VerticalSteps currentStep={0} stepLabels={steps} />
</div>
```

**Đúng:**

```tsx
// ✅ Override sizing nội bộ qua className
<VerticalSteps
  currentStep={0}
  stepLabels={steps}
  className="min-w-0 max-w-full w-[12.5rem]"
/>
```

---

### RULE-VSTEPS-05: Giữ độ dài `stepLabels` ổn định — không thêm hoặc xóa steps có điều kiện

Các đường kết nối được render dựa trên `stepLabels.length`. Thêm hoặc xóa steps giữa luồng khiến chuỗi kết nối trực quan nhảy.

**Sai:**

```tsx
// ❌ stepLabels thay đổi độ dài dựa trên role — đường kết nối dịch chuyển không mong đợi
const steps = isAdmin
  ? ["Setup", "Permissions", "Review"]
  : ["Setup", "Review"];

<VerticalSteps currentStep={current} stepLabels={steps} />;
```

**Đúng:**

```tsx
// ✅ Giữ steps ổn định; sử dụng các flows riêng nếu số lượng steps thực sự khác nhau
const STEPS = ["Setup", "Permissions", "Review"];

<VerticalSteps currentStep={current} stepLabels={STEPS} />;
```
