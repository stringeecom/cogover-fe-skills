## Form với react-hook-form và Controlled Components

### ⚠️ NGUYÊN TẮC BẮT BUỘC: LUÔN dùng Controlled Components

Khi tạo bất kỳ input nào trong `react-hook-form`, **BẮT BUỘC** sử dụng Controlled Components (các component có prefix `Controlled*` từ `@stringeecom/ui-kit` hoặc tự tạo theo RULE-FORM-04). **TUYỆT ĐỐI KHÔNG** dùng uncontrolled components với `register()` hoặc `ref`.

```tsx
// ❌ NGHIÊM CẤM — Uncontrolled component với register
<input {...methods.register("name")} />
<TextField {...methods.register("email")} />

// ❌ NGHIÊM CẤM — Dùng Controller trực tiếp khi đã có Controlled component sẵn
<Controller name="name" render={({ field }) => <TextField {...field} />} />

// ✅ BẮT BUỘC — Dùng Controlled component
<ControlledTextField name="name" label="Tên" />
<ControlledSelect name="status" options={statusOptions} />
```

**Lý do:**
- Đảm bảo tính nhất quán trong toàn bộ codebase
- Controlled components đã xử lý sẵn error display, `onBlur`, `onChange`, typing
- Giảm boilerplate code, tránh lỗi lặp lại
- Nếu chưa có Controlled component phù hợp → tạo mới theo RULE-FORM-04, **KHÔNG** dùng `register()` hay `Controller` trực tiếp

---

### RULE-FORM-01: Luôn dùng `useForm` + `FormProvider`

```tsx
import { FormProvider, useForm } from "react-hook-form";

interface FormData { name: string; email: string; }

const methods = useForm<FormData>({ defaultValues: { name: "", email: "" } });

<FormProvider {...methods}>
  <form onSubmit={methods.handleSubmit(onSubmit)}>
    {/* Controlled components */}
  </form>
</FormProvider>
```

---

### RULE-FORM-02: Dùng Controlled component — không dùng `<Controller />` trực tiếp

```tsx
// ❌
<Controller control={methods.control} name="name"
  render={({ field, fieldState: { error } }) => (
    <TextField {...field} error={!!error} helperText={error?.message} />
  )}
/>

// ✅
<ControlledTextField name="name" label="Tên" />
<ControlledSelect name="status" options={statusOptions} />
```

---

### RULE-FORM-03: Kiểm tra kiểu `value` của Controlled component trước khi dùng

```tsx
// ✅ Kiểu dữ liệu khớp với Controlled component
interface FormData {
  name: string;                    // ControlledTextField
  age: number | null;              // ControlledInputNumber
  assignees: string[];             // ControlledRecordMultipleSelect
  startDate: Dayjs | null;         // ControlledDatePicker
  isActive: boolean;               // ControlledCheckbox
  tags: TagsInputValue<string>[];  // ControllerTagInput
}
```

---

### RULE-FORM-04: Tạo Controlled component mới nếu chưa có sẵn

```tsx
import { useFormContext, Controller } from "react-hook-form";

interface ControlledSwitchProps extends Omit<SwitchProps, "checked" | "onChange"> {
  name: string;
}

const ControlledSwitch = ({ name, ...props }: ControlledSwitchProps) => {
  const { control } = useFormContext();
  return (
    <Controller control={control} name={name}
      render={({ field: { value, onChange, onBlur } }) => (
        <Switch {...props} checked={value} onChange={(e) => onChange(e.target.checked)} onBlur={onBlur} />
      )}
    />
  );
};
```

---

### RULE-FORM-05: Dùng `yup` + `yupResolver` để validate

```tsx
import * as yup from "yup";
import { yupResolver } from "@hookform/resolvers/yup";

const schema = yup.object({
  name: yup.string().required("Vui lòng nhập tên").trim(),
  email: yup.string().required().email("Email không hợp lệ"),
  age: yup.number().nullable().min(1),
  assignees: yup.array().of(yup.string()).min(1),
});

const methods = useForm<FormData>({ defaultValues, resolver: yupResolver(schema), mode: "onChange" });
```

---

### Danh sách Controlled Components

| Component | Value Type |
|-----------|-----------|
| `ControlledTextField` | `string` |
| `ControlledInputNumber` | `number \| null` |
| `ControlledSelect` | Generic `T` |
| `ControlledCheckbox` | `boolean` |
| `ControlledRadioGroup` | `string` |
| `ControlledDatePicker` | `Dayjs \| null` |
| `ControlledDateRangePicker` | `[Dayjs, Dayjs]` |
| `ControlledSearchInput` | `string` |
| `ControlledCKEditor` | `string` |
| `ControlledObjectSelect` | `string \| null` |
| `ControlledRecordSelect` | `string[]` |
| `ControlledRecordMultipleSelect` | `string[]` |
| `ControlledRoleMultipleSelect` | `string[]` |
| `ControllerTagInput` | `TagsInputValue<T>[]` |
| `ControlledPersonnelMultipleSelect` | **DEPRECATED** — dùng `ControlledRecordMultipleSelect` với `objectSlug="personnel"` |
