---
title: Cách sử dụng component Steps trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến highlighting step sai, trạng thái active off-by-one hoặc navigation bị lỗi
tags: steps, wizard, progress, ui-kit
---

## Cách sử dụng component Steps trong ui-kit

Đường dẫn component: `ui-kit/src/components/Steps`

Props: `stepLabels` (bắt buộc), `currentStepIndex`, `className`.

---

### RULE-STEPS-01: `currentStepIndex` là 0-based — không bao giờ truyền giá trị 1-based

`currentStepIndex` mặc định là `-1` (không có steps active). Để kích hoạt step đầu tiên, truyền `0`, không phải `1`. Truyền giá trị 1-based làm dịch chuyển tất cả highlighting một step về phía trước.

**Sai:**

```tsx
// ❌ 1-based — step 2 được highlight thay vì step 1
<Steps stepLabels={["Account", "Settings", "Review"]} currentStepIndex={1} />
```

**Đúng:**

```tsx
// ✅ 0-based — step 1 được highlight đúng
<Steps stepLabels={["Account", "Settings", "Review"]} currentStepIndex={0} />

// ✅ Không có steps active — bỏ qua hoặc truyền -1 rõ ràng
<Steps stepLabels={["Account", "Settings", "Review"]} />
<Steps stepLabels={["Account", "Settings", "Review"]} currentStepIndex={-1} />
```

---

### RULE-STEPS-02: Tất cả steps đến và bao gồm `currentStepIndex` đều được highlight — không có sự phân biệt "completed vs current"

`isActiveStep = index <= currentStepIndex`, vì vậy mọi step trước step hiện tại đều được styled giống hệt với step hiện tại (cả hai đều dùng màu primary). Không mong đợi sự khác biệt trực quan giữa các past và current steps.

**Sai:**

```tsx
// ❌ Giả sử chỉ step hiện tại được highlight — nhưng steps 0 và 1 cả hai đều xuất hiện active
<Steps stepLabels={["Account", "Settings", "Review"]} currentStepIndex={1} />
// Step 1 ("Account") và Step 2 ("Settings") cả hai đều màu primary
```

**Đúng:**

```tsx
// ✅ Hiểu rằng tất cả steps đến currentStepIndex xuất hiện active (màu primary)
// Nếu bạn cần phân biệt "completed" vs "current", render stepLabels tùy chỉnh với icons
<Steps
  stepLabels={[
    <CheckIcon />, // completed
    "Settings", // current
    "Review", // upcoming
  ]}
  currentStepIndex={1}
/>
```

---

### RULE-STEPS-03: Đường kết nối kích hoạt giữa hai steps liền kề đang active — không chỉ ở step hiện tại

Đường giữa step `i` và step `i+1` chuyển màu primary khi `i <= currentStepIndex - 1`, có nghĩa là đường chỉ được filled khi cả hai steps xung quanh nó đều active.

**Mô hình tinh thần đúng:**

```
currentStepIndex = 0  →  step 1 active,  không có đường active
currentStepIndex = 1  →  step 1+2 active, đường giữa 1-2 active
currentStepIndex = 2  →  step 1+2+3 active, đường giữa 1-2 và 2-3 active
```

---

### RULE-STEPS-04: Không thêm click handlers trên chính `Steps` — navigation phải được quản lý từ bên ngoài

`Steps` là một component chỉ hiển thị. Nó không có `onClickStep` hoặc navigation callback. Điều khiển thay đổi steps qua state riêng của bạn và các nút điều khiển.

**Sai:**

```tsx
// ❌ Steps không có prop onClick — cái này không làm gì cả
<Steps
  stepLabels={steps}
  currentStepIndex={current}
  onClick={(i) => setCurrent(i)}
/>
```

**Đúng:**

```tsx
// ✅ Quản lý navigation từ bên ngoài
const [current, setCurrent] = useState(0);

<Steps stepLabels={steps} currentStepIndex={current} />

<Button onClick={() => setCurrent((s) => Math.min(s + 1, steps.length - 1))}>
  Next
</Button>
<Button onClick={() => setCurrent((s) => Math.max(s - 1, 0))}>
  Back
</Button>
```

---

### RULE-STEPS-05: Đảm bảo container đủ rộng — step labels được positioned absolutely và có thể overlap

Mỗi step label sử dụng `position: absolute` được canh giữa bên dưới vòng tròn của nó. Trong một container hẹp, labels từ các steps liền kề sẽ overlap. Luôn cung cấp cho `Steps` đủ không gian ngang tương ứng với số lượng steps.

**Sai:**

```tsx
// ❌ Fixed narrow width với nhiều steps — labels overlap
<div className="w-[12.5rem]">
  <Steps
    stepLabels={["Account Info", "Preferences", "Notifications", "Review"]}
    currentStepIndex={0}
  />
</div>
```

**Đúng:**

```tsx
// ✅ Để container fill độ rộng có sẵn
<div className="w-full">
  <Steps stepLabels={["Account Info", "Preferences", "Notifications", "Review"]} currentStepIndex={0} />
</div>

// ✅ Hoặc giới hạn với độ rộng cho đủ chỗ cho mỗi step (ít nhất ~100px mỗi step)
<div className="w-[37.5rem]">
  <Steps stepLabels={["Account Info", "Preferences", "Notifications", "Review"]} currentStepIndex={0} />
</div>
```

---

### RULE-STEPS-06: Giữ độ dài `stepLabels` ổn định — không thêm hoặc xóa steps có điều kiện

Số lượng vòng tròn và đường kết nối được suy ra trực tiếp từ `stepLabels.length`. Thêm hoặc xóa steps giữa luồng khiến UI nhảy và phá vỡ chỉ báo tiến trình trực quan.

**Sai:**

```tsx
// ❌ stepLabels thay đổi độ dài dựa trên điều kiện — đường kết nối nhảy
const steps = isAdmin
  ? ["Account", "Permissions", "Review"]
  : ["Account", "Review"];

<Steps stepLabels={steps} currentStepIndex={current} />;
```

**Đúng:**

```tsx
// ✅ Giữ mảng steps ổn định; sử dụng các flows riêng nếu số lượng steps thực sự khác nhau
const STEPS = ["Account", "Permissions", "Review"];

<Steps stepLabels={STEPS} currentStepIndex={current} />;
```
