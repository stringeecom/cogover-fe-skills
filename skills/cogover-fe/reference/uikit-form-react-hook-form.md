---
title: Cách tạo form với react-hook-form và controlled components
impact: HIGH
impactDescription: Tạo form sai pattern dẫn đến state không đồng bộ, validation không hoạt động, và code khó bảo trì
tags: form, react-hook-form, controlled, yup, validation, FormProvider
---

## Cách tạo form với react-hook-form và controlled components

### RULE-FORM-01: Luôn dùng react-hook-form với FormProvider

Khi tạo form, luôn dùng `useForm` để tạo methods và bọc form trong `FormProvider`.

```tsx
import { FormProvider, useForm } from "react-hook-form";

interface FormData {
  name: string;
  email: string;
}

const methods = useForm<FormData>({
  defaultValues: {
    name: "",
    email: "",
  },
});

<FormProvider {...methods}>
  <form onSubmit={methods.handleSubmit(onSubmit)}>
    {/* Controlled components ở đây */}
  </form>
</FormProvider>
```

### RULE-FORM-02: Luôn dùng Controlled component, không dùng Controller trực tiếp

Các element trong form phải sử dụng Controlled component có sẵn và truyền `name` vào. KHÔNG dùng `<Controller />` trực tiếp trong code form.

**Sai:**
```tsx
// ❌ Dùng Controller trực tiếp trong form
<Controller
  control={methods.control}
  name="name"
  render={({ field, fieldState: { error } }) => (
    <TextField {...field} error={!!error} helperText={error?.message} />
  )}
/>
```

**Đúng:**
```tsx
// ✅ Dùng Controlled component
<ControlledTextField name="name" label="Tên" />
<ControlledSelect name="status" options={statusOptions} />
<ControlledDatePicker name="startDate" />
```

### RULE-FORM-03: Kiểm tra interface của value trước khi dùng Controlled component

Trước khi sử dụng bất kỳ Controlled component nào, phải đọc code để kiểm tra kiểu dữ liệu của `value` mà component đó quản lý, từ đó thiết kế `FormData` type cho chính xác.

**Ví dụ:** `ControlledRecordMultipleSelect` quản lý value kiểu `string[]`, vậy field tương ứng trong FormData phải là `string[]`.

```tsx
// ✅ Kiểu dữ liệu khớp với value của Controlled component
interface FormData {
  name: string;                    // ControlledTextField -> string
  age: number | null;              // ControlledInputNumber -> number | null
  status: string;                  // ControlledSelect -> phụ thuộc getValue
  assignees: string[];             // ControlledRecordMultipleSelect -> string[]
  startDate: Dayjs | null;         // ControlledDatePicker -> Dayjs | null
  isActive: boolean;               // ControlledCheckbox -> boolean
  tags: TagsInputValue<string>[];  // ControllerTagInput -> TagsInputValue<T>[]
}
```

### RULE-FORM-04: Tự tạo Controlled component nếu chưa có sẵn

Nếu không tìm thấy Controlled component cho UI component cần dùng, phải tạo mới một Controlled component theo cùng pattern, KHÔNG dùng `<Controller />` trực tiếp.

**Pattern tạo Controlled component mới:**
```tsx
import { useFormContext, Controller } from "react-hook-form";

interface ControlledSwitchProps extends Omit<SwitchProps, "checked" | "onChange"> {
  name: string;
}

const ControlledSwitch = ({ name, ...props }: ControlledSwitchProps) => {
  const { control } = useFormContext();

  return (
    <Controller
      control={control}
      name={name}
      render={({ field: { value, onChange, onBlur } }) => (
        <Switch
          {...props}
          checked={value}
          onChange={(e) => onChange(e.target.checked)}
          onBlur={onBlur}
        />
      )}
    />
  );
};
```

### RULE-FORM-05: Luôn hỏi lại logic validate nếu user chưa cung cấp

Khi được yêu cầu tạo form mà user chưa mô tả logic validate, **phải hỏi lại** trước khi code. Ví dụ:
- Field nào bắt buộc?
- Giới hạn độ dài (min/max) của text field?
- Giới hạn giá trị (min/max) của number field?
- Format đặc biệt (email, phone, URL)?
- Validation phụ thuộc giữa các field?

### RULE-FORM-06: Dùng yup tạo schema và yupResolver để validate

Sau khi có thông tin validate, phải dùng `yup` để tạo schema và `yupResolver` để kết nối với react-hook-form.

```tsx
import * as yup from "yup";
import { yupResolver } from "@hookform/resolvers/yup";

const schema = yup.object({
  name: yup.string().required("Vui lòng nhập tên").trim(),
  email: yup.string().required("Vui lòng nhập email").email("Email không hợp lệ"),
  age: yup.number().nullable().min(1, "Tuổi phải lớn hơn 0"),
  assignees: yup.array().of(yup.string()).min(1, "Vui lòng chọn ít nhất 1 người"),
});

const methods = useForm<FormData>({
  defaultValues,
  resolver: yupResolver(schema),
  mode: "onChange",
});
```

**Custom validation với `.test()`:**
```tsx
const schema = yup.object({
  endDate: yup.mixed().nullable().test({
    message: "Ngày kết thúc phải sau ngày bắt đầu",
    test: (value, context) => {
      const startDate = context.parent.startDate;
      if (!value || !startDate) return true;
      return dayjs(value).isAfter(dayjs(startDate));
    },
  }),
});
```

### Danh sách Controlled Components có sẵn

| Component | Value Type | Mô tả |
|-----------|-----------|--------|
| `ControlledTextField` | `string` | Text input, hỗ trợ `acceptRegex` |
| `ControlledInputNumber` | `number \| null` | Input số |
| `ControlledSelect` | Generic `T` | Select với custom `getValue`/`getLabel` |
| `ControlledCheckbox` | `boolean` | Checkbox, hỗ trợ `label` |
| `ControlledRadioGroup` | `string` | Nhóm radio button |
| `ControlledDatePicker` | `Dayjs \| null` | Chọn ngày, hỗ trợ `transforms`/`restores` |
| `ControlledDatePickerRelative` | `Dayjs \| null` + TimeAgo | Chọn ngày với thời gian tương đối |
| `ControlledDateRangePicker` | `[Dayjs, Dayjs]` | Chọn khoảng ngày |
| `ControlledSearchInput` | `string` | Input tìm kiếm |
| `ControlledCKEditor` | `string` | Rich text editor (CKEditor) |
| `ControlledSlateEditor` | SlateEditor value | Rich text editor (Slate) |
| `ControlledObjectSelect` | `string \| null` | Chọn object |
| `ControlledRecordSelect` | `string[]` | Chọn record theo `objectSlug` |
| `ControlledRecordMultipleSelect` | `string[]` | Chọn nhiều record |
| `ControlledAssignPersonnelSelect` | personnel value | Chọn nhân viên |
| `ControlledRoleMultipleSelect` | `string[]` | Chọn role |
| `ControllerTagInput` | `TagsInputValue<T>[]` | Input tags |
| `ControlledPersonnelMultipleSelect` | `string[]` | **DEPRECATED** — dùng `ControlledRecordMultipleSelect` với `objectSlug="personnel"` |
