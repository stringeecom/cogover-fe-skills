---
title: Cách sử dụng component HorizontalSteps trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến hiển thị step sai lệch, trạng thái active off-by-one hoặc navigation bị lỗi
tags: steps, stepper, progress, wizard, ui-kit
---

## Cách sử dụng component HorizontalSteps trong ui-kit

Đường dẫn component: `ui-kit/src/components/HorizontalSteps`

Props: `currentStep`, `stepLabels`, `onClickStep`, `isCanClick`, `className`, `classNameIcon`.

### RULE-HSTEPS-01: `currentStep` là 0-based — không truyền giá trị 1-based

`currentStep` là zero-based index. Step đầu tiên là `0`, step thứ hai là `1`, v.v. Truyền giá trị 1-based làm dịch chuyển tất cả các trạng thái step đi một.

**Sai:**

```tsx
// ❌ 1-based: step 1 hiển thị là "current" nhưng index 0 đã là "completed"
<HorizontalSteps currentStep={1} stepLabels={["Bước 1", "Bước 2", "Bước 3"]} />
// khi ở step đầu tiên
```

**Đúng:**

```tsx
// ✅ 0-based
const [step, setStep] = useState(0); // step đầu tiên
<HorizontalSteps
  currentStep={step}
  stepLabels={["Bước 1", "Bước 2", "Bước 3"]}
/>;
```

### RULE-HSTEPS-02: Luôn set `isCanClick` khi `onClickStep` bật navigation

`isCanClick` thêm `cursor-pointer` vào các step indicators. Không có nó, click hoạt động âm thầm không có affordance trực quan. Luôn ghép `isCanClick` với `onClickStep`.

**Sai:**

```tsx
// ❌ Click hoạt động nhưng không có phản hồi cursor
<HorizontalSteps currentStep={step} stepLabels={labels} onClickStep={setStep} />
```

**Đúng:**

```tsx
// ✅
<HorizontalSteps
  currentStep={step}
  stepLabels={labels}
  isCanClick
  onClickStep={setStep}
/>
```

### RULE-HSTEPS-03: Không truyền `onClickStep` khi các steps không thể navigate

`onClickStep` kích hoạt on click ngay cả khi `isCanClick` là `false`. Nếu steps là các chỉ báo tiến trình chỉ đọc, bỏ qua `onClickStep` hoàn toàn.

**Sai:**

```tsx
// ❌ onClickStep không có ý định navigate gây thay đổi step ngẫu nhiên
<HorizontalSteps currentStep={step} stepLabels={labels} onClickStep={setStep} />
```

**Đúng:**

```tsx
// ✅ Chỉ đọc — bỏ qua onClickStep
<HorizontalSteps currentStep={step} stepLabels={labels} />
```

### RULE-HSTEPS-04: Giữ độ dài `stepLabels` ổn định — không thêm/xóa steps có điều kiện

Các đường kết nối được tính từ độ dài mảng label. Thêm hoặc xóa labels giữa luồng gây layout nhảy và step numbers không khớp.

**Sai:**

```tsx
// ❌ Mảng labels thay đổi độ dài dựa trên điều kiện
const labels = ["Bước 1", "Bước 2"];
if (showExtraStep) labels.push("Bước 3");
<HorizontalSteps currentStep={step} stepLabels={labels} />;
```

**Đúng:**

```tsx
// ✅ Mảng label cố định — kiểm soát visibility qua logic currentStep thay vào đó
const labels = ["Bước 1", "Bước 2", "Bước 3"];
<HorizontalSteps currentStep={step} stepLabels={labels} />;
```

### RULE-HSTEPS-05: Sử dụng `classNameIcon` để style các vòng tròn step, không dùng `className`

`className` áp dụng cho wrapper bên ngoài. Để tùy chỉnh các vòng tròn số step (size, border, v.v.), sử dụng `classNameIcon`.

**Sai:**

```tsx
// ❌ className target wrapper, không phải các vòng tròn
<HorizontalSteps
  currentStep={step}
  stepLabels={labels}
  className="[&_.circle]:w-10"
/>
```

**Đúng:**

```tsx
// ✅
<HorizontalSteps
  currentStep={step}
  stepLabels={labels}
  classNameIcon="w-10 h-10"
/>
```
