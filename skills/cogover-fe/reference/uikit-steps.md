## Steps Component (`uikit-steps`)

Đường dẫn: `ui-kit/src/components/Steps`

Component chỉ hiển thị (không có state nội bộ, không có `onClickStep`). Props: `stepLabels` (bắt buộc), `currentStepIndex` (mặc định -1), `className`.

Tất cả steps đến và bao gồm `currentStepIndex` đều được highlight (màu primary). Đường kết nối giữa step `i` và `i+1` filled khi `i <= currentStepIndex - 1`.

---

### RULE-STEPS-01: `currentStepIndex` là 0-based — mặc định -1 (không có steps active)

```tsx
// ❌ 1-based — step 2 được highlight thay vì step 1
<Steps stepLabels={["Account", "Settings", "Review"]} currentStepIndex={1} />

// ✅
<Steps stepLabels={["Account", "Settings", "Review"]} currentStepIndex={0} />  // Step 1 active
<Steps stepLabels={["Account", "Settings", "Review"]} />                        // Không có steps active
```

---

### RULE-STEPS-02: Navigation phải được quản lý từ bên ngoài — không có `onClickStep`

```tsx
// ❌ Steps không có prop onClick
<Steps stepLabels={steps} currentStepIndex={current} onClick={(i) => setCurrent(i)} />

// ✅
const [current, setCurrent] = useState(0);
<Steps stepLabels={steps} currentStepIndex={current} />
<Button onClick={() => setCurrent(s => Math.min(s + 1, steps.length - 1))}>Next</Button>
<Button onClick={() => setCurrent(s => Math.max(s - 1, 0))}>Back</Button>
```

---

### RULE-STEPS-03: Đảm bảo container đủ rộng — labels positioned absolutely, có thể overlap

```tsx
// ❌ Container hẹp — labels overlap
<div className="w-[12.5rem]">
  <Steps stepLabels={["Account Info", "Preferences", "Notifications", "Review"]} currentStepIndex={0} />
</div>

// ✅ Cho đủ không gian (ít nhất ~100px mỗi step)
<div className="w-full">
  <Steps stepLabels={["Account Info", "Preferences", "Notifications", "Review"]} currentStepIndex={0} />
</div>
```

---

### RULE-STEPS-04: Giữ độ dài `stepLabels` ổn định — không thêm/xóa steps có điều kiện

```tsx
// ❌ stepLabels thay đổi độ dài — đường kết nối nhảy
const steps = isAdmin ? ["Account", "Permissions", "Review"] : ["Account", "Review"];

// ✅ Dùng các flows riêng nếu số lượng steps thực sự khác nhau
const STEPS = ["Account", "Permissions", "Review"];
<Steps stepLabels={STEPS} currentStepIndex={current} />
```
