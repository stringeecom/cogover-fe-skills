---
title: Cách sử dụng component FormBuilder trong ui-kit
impact: HIGH
impactDescription: Sử dụng sai dẫn đến form không render, validation không hoạt động, submit thất bại, và mất dữ liệu khi edit record
tags: form-builder, layout, record, react-hook-form, validation, ui-kit
---

## Cách sử dụng component FormBuilder trong ui-kit

Đường dẫn component: `ui-kit/src/components/FormBuilder`
Example: `ui-kit/src/example/FormBuilder/index.tsx`

Props bắt buộc: `items`, `objectSlug`, `objectId`, `layoutSlug`.
Props tuỳ chọn: `layoutId`, `record`, `defaultValues`, `detailForm`, `layoutType`, `canEdit`, `className`, `formId`, `richEditorStickyToolbar`, `onEditableSubmit`, `onChangeFieldFormBuilder`, `readOnlyFieldSlugs`, `forceReadOnlySlugs`, `disabledLayoutRules`, `disabledUpdateFormValueByRecordDetail`, `disabledTransitionRule`, `isUserTaskForm`, `instanceStatus`, `processId`, `nodeScreenId`, `pageSettings`, `updateRecordMode`, `additionalOptionsForLookupFields`, `screen`, `groupSlots`, `fieldSlots`, `renderField`, `onClickButtonGroup`, `onClickWorkflowButton`, `onClickAddLookupButton`, `onEditableCancel`, `onBlurEditableFormItem`, `onChangeLayout`, `onChangeScreen`, `onReadyFirstLayoutRuleRun`, `isLoadingButton`, `isLoadingButtonGroup`, `checkDisabledButtonGroup`, `checkDisabledWorkflowButton`, `handleClickButtonGroupProps`, `buttonGroupProps`, `personnelElementProps`.

Giá trị mặc định: `detailForm=false`, `isUserTaskForm=false`.

---

### RULE-FORM-BUILDER-01: FormBuilder bắt buộc phải nằm trong FormProvider của react-hook-form

`FormBuilder` không tự tạo form instance. Parent component phải tạo `useForm` và bọc `FormBuilder` trong `<FormProvider>`. FormBuilder đọc form state từ context.

**Sai:**

```tsx
// ❌ FormBuilder không nằm trong FormProvider — form không hoạt động
<FormBuilder
  ref={formBuilderRef}
  items={layout}
  objectSlug={objectSlug}
  objectId={objectId}
  layoutSlug={layoutSlug}
/>
```

**Đúng:**

```tsx
// ✅ Bọc trong FormProvider
const methods = useForm<Record<string, unknown>>({
  mode: "onChange",
  defaultValues: values,
  resolver: yupResolver(object(schema)),
});

<FormProvider {...methods}>
  <FormBuilder
    ref={formBuilderRef}
    items={layout}
    objectSlug={objectSlug}
    objectId={objectId}
    layoutSlug={layoutSlug}
  />
</FormProvider>
```

---

### RULE-FORM-BUILDER-02: Phải sử dụng useCreateDefaultValues và useCreateSchema để tạo defaultValues và schema từ layout

Không tự tạo defaultValues hoặc schema thủ công. Sử dụng `useCreateDefaultValues` để tạo giá trị mặc định từ layout tree (và record nếu edit mode), và `useCreateSchema` để tạo yup schema từ layout. Truyền schema vào `yupResolver` của `useForm`.

**Sai:**

```tsx
// ❌ Tự tạo defaultValues thủ công — thiếu các field từ layout
const methods = useForm({
  defaultValues: { name: "", email: "" },
});
```

**Đúng:**

```tsx
// ✅ Sử dụng useCreateDefaultValues và useCreateSchema
const { createDefaultValues } = useCreateDefaultValues();
const { schema } = useCreateSchema({ layout, formId: formId.current });

const values = useMemo(() => {
  if (editMode && record) {
    return createDefaultValues(layout, record);
  }
  return createDefaultValues(layout);
}, [createDefaultValues, layout, record]);

const methods = useForm<Record<string, unknown>>({
  mode: "onChange",
  defaultValues: values,
  resolver: yupResolver(object(schema)),
});
```

---

### RULE-FORM-BUILDER-03: Phải truyền ref cho FormBuilder và sử dụng useSubmitFormBuilder để submit

`FormBuilder` sử dụng `forwardRef` để expose `FormBuilderRef`. Phải tạo ref và truyền vào `FormBuilder`, sau đó truyền ref này cho `useSubmitFormBuilder` để xử lý submit. Không tự gọi `methods.handleSubmit` trực tiếp để submit form builder.

**Sai:**

```tsx
// ❌ Tự gọi handleSubmit — không xử lý được nested children, body construction, và special objects
<Button onClick={methods.handleSubmit(onSubmit)}>Submit</Button>
```

**Đúng:**

```tsx
// ✅ Sử dụng useSubmitFormBuilder với ref
const formBuilderRef = useRef<FormBuilderRef>(null);

const { handleSubmit, isLoading: isSubmitting } = useSubmitFormBuilder({
  ref: formBuilderRef,
  apiUrlSubmit: layoutResponse?.pageSettings?.settingPage?.apiUrlSubmit,
});

<FormBuilder ref={formBuilderRef} items={layout} ... />
<Button loading={isSubmitting} onClick={() => handleSubmit()}>Submit</Button>
```

---

### RULE-FORM-BUILDER-04: Sử dụng formId từ uuid, không tự tạo string bất kỳ

`formId` được dùng nội bộ để định danh form instance (dùng cho schema, error handling, events). Phải tạo bằng `uuid()` và giữ trong `useRef` để không thay đổi giữa các lần render. Truyền vào cả `useCreateSchema` và `FormBuilder`.

**Sai:**

```tsx
// ❌ formId thay đổi mỗi lần render — gây re-render liên tục và schema không khớp
const formId = uuid();
```

**Đúng:**

```tsx
// ✅ formId ổn định qua useRef
import { v4 as uuid } from "uuid";

const formId = useRef(uuid());

const { schema } = useCreateSchema({ layout, formId: formId.current });

<FormBuilder formId={formId.current} ref={formBuilderRef} items={layout} ... />
```

---

### RULE-FORM-BUILDER-05: Phân biệt rõ layoutType và detailForm cho create mode và edit mode

- **Create mode**: `layoutType="create"`, `detailForm={false}` (mặc định), không truyền `record`.
- **Edit/Detail mode**: `layoutType="view"`, `detailForm={true}`, truyền `record`, và cung cấp `onEditableSubmit` để xử lý inline save khi blur.

Không set `detailForm={true}` mà không truyền `record` và `onEditableSubmit`.

**Sai:**

```tsx
// ❌ detailForm=true nhưng không có record và onEditableSubmit
<FormBuilder
  items={layout}
  objectSlug={objectSlug}
  objectId={objectId}
  layoutSlug={layoutSlug}
  detailForm={true}
  layoutType="create"
/>
```

**Đúng:**

```tsx
// ✅ Create mode
<FormBuilder
  items={layout}
  objectSlug={objectSlug}
  objectId={objectId}
  layoutSlug={layoutSlug}
  layoutType="create"
/>

// ✅ Edit/Detail mode
<FormBuilder
  items={layout}
  objectSlug={objectSlug}
  objectId={objectId}
  layoutSlug={layoutSlug}
  layoutType="view"
  detailForm={true}
  record={record}
  defaultValues={values}
  updateRecordMode={updateRecordMode}
  onEditableSubmit={async (data) => {
    try {
      await updateRecord({
        objectId: objectDetail.id,
        id: record.id,
        data,
      });
      return true;
    } catch {
      return false;
    }
  }}
/>
```

---

### RULE-FORM-BUILDER-06: Truyền layoutId để bật layout rules — không truyền khi không cần

`layoutId` kết hợp `layoutSlug` bật Layout Rule engine (`LayoutRuleProvider`). Khi không truyền `layoutId`, không có layout rule nào được evaluate. Sử dụng `disabledLayoutRules={true}` để force-skip rules ngay cả khi đã truyền `layoutId`.

**Sai:**

```tsx
// ❌ Truyền layoutId rỗng — gây lỗi khi LayoutRuleProvider có giá trị
<FormBuilder
  items={layout}
  objectSlug={objectSlug}
  objectId={objectId}
  layoutSlug={layoutSlug}
  layoutId=""
/>
```

**Đúng:**

```tsx
// ✅ Truyền layoutId từ API response để bật layout rules
<FormBuilder
  items={layout}
  objectSlug={objectSlug}
  objectId={objectId}
  layoutSlug={activeLayoutSlug}
  layoutId={activeLayoutId}
/>

// ✅ Tắt layout rules khi không cần
<FormBuilder
  items={layout}
  objectSlug={objectSlug}
  objectId={objectId}
  layoutSlug={layoutSlug}
  disabledLayoutRules={true}
/>
```

---

### RULE-FORM-BUILDER-07: Sử dụng useGetLayout để lấy layout data — không tự build layout tree

`items` (layout) là cây dữ liệu từ server. Phải dùng hook `useGetLayout` để fetch layout data, không tự tạo cấu trúc `FormBuilderLayoutRow[]` thủ công. Cấu trúc layout là: `Row → Column → Section → Tab → Group → Component (field)`.

**Sai:**

```tsx
// ❌ Tự tạo layout thủ công — thiếu metadata, sai cấu trúc
const layout = [{ id: "1", type: "layoutRow", children: [] }];
```

**Đúng:**

```tsx
// ✅ Sử dụng useGetLayout để fetch layout từ API
const { layout, isLoading, activeLayoutSlug, activeLayoutId, layoutResponse } = useGetLayout({
  functionLabel: editMode ? "VIEW" : "ADD",
  objectSlug,
  layoutId,
});

if (isLoading || !layoutResponse) return null;

<FormBuilder items={layout} layoutSlug={activeLayoutSlug} layoutId={activeLayoutId} ... />
```

---

### RULE-FORM-BUILDER-08: Sử dụng setFormBuilderFieldError để hiển thị lỗi từ API response — không tự set error thủ công

Khi API trả về lỗi validation (error meta), sử dụng hàm `setFormBuilderFieldError` với `formId` và `errors` từ API response. Không tự gọi `methods.setError` cho từng field.

**Sai:**

```tsx
// ❌ Tự set error cho từng field — thiếu logic mapping từ API response
try {
  await updateRecord(data);
} catch (error) {
  methods.setError("name", { message: "Tên đã tồn tại" });
  methods.setError("email", { message: "Email không hợp lệ" });
}
```

**Đúng:**

```tsx
// ✅ Sử dụng setFormBuilderFieldError
import { setFormBuilderFieldError } from "@stringeecom/ui-kit";

const { mutateAsync: updateRecord } = useUpdateRecord({
  apiUrlSubmit,
  onError: (error) => {
    const errorMeta = objectPath.get(error, "response.data.body.meta", {});
    setFormBuilderFieldError({
      formId: formId.current,
      errors: errorMeta,
    });
  },
});
```

---

### RULE-FORM-BUILDER-09: Sử dụng useFormBuilderButton và useHandleClickButtonGroup để xử lý button group — không tự build logic

Khi cần hiển thị và xử lý các button group (tạo/xoá/cập nhật/export record...), sử dụng `useFormBuilderButton` để quản lý trạng thái modal và `useHandleClickButtonGroup` để xử lý sự kiện click. Không tự build logic xử lý button group.

**Sai:**

```tsx
// ❌ Tự build logic xử lý button — thiếu nhiều case và modal state
const handleClick = (button) => {
  if (button.type === "create") createRecord();
  if (button.type === "delete") deleteRecord();
};
```

**Đúng:**

```tsx
// ✅ Sử dụng useFormBuilderButton và useHandleClickButtonGroup
const {
  isOpenButtonRecordModal,
  setIsOpenButtonRecordModal,
  isOpenButtonDeleteRecordModal,
  setIsOpenButtonDeleteRecordModal,
  selectedButtonGroup,
  setSelectedButtonGroup,
  // ... các modal state khác
} = useFormBuilderButton();

const { handleClickButtonGroup } = useHandleClickButtonGroup({
  onClick: (buttonResponse) => setSelectedButtonGroup(buttonResponse),
  onCreateRecord: () => setIsOpenButtonRecordModal(true),
  onUpdateRecord: () => setIsOpenButtonRecordModal(true),
  onDeleteRecord: () => setIsOpenButtonDeleteRecordModal(true),
  onReset: () => { /* reset logic */ },
  onCancel: () => { /* cancel logic */ },
  // ... các handler khác
});
```

---

### RULE-FORM-BUILDER-10: Sử dụng groupSlots và fieldSlots để custom render — không override CSS nội bộ

Khi cần tuỳ chỉnh render cho một group hoặc field cụ thể, sử dụng prop `groupSlots` hoặc `fieldSlots` với `condition` function để xác định group/field cần custom và `render` function để trả về React node. Không override CSS selector nội bộ của FormBuilder.

**Sai:**

```tsx
// ❌ Override CSS nội bộ — dễ vỡ khi component cập nhật
<FormBuilder
  className={cx("[&_.form-builder-group]:hidden")}
  items={layout}
  ...
/>
```

**Đúng:**

```tsx
// ✅ Sử dụng fieldSlots để custom render
<FormBuilder
  items={layout}
  objectSlug={objectSlug}
  objectId={objectId}
  layoutSlug={layoutSlug}
  fieldSlots={[
    {
      condition: (field) => field.slug === "custom_field",
      render: (field) => <CustomFieldComponent field={field} />,
    },
  ]}
  groupSlots={[
    {
      condition: (group) => group.slug === "custom_group",
      render: (group) => <CustomGroupComponent group={group} />,
    },
  ]}
/>
```

---

### RULE-FORM-BUILDER-11: Sử dụng LayoutRuleProvider khi cần kiểm tra change layout trước khi render FormBuilder

Khi cần kiểm tra layout rule có thay đổi layout hay không trước khi render FormBuilder (tránh nhấp nháy giao diện), bọc FormBuilder trong `LayoutRuleProvider` với `onlyChangeLayout` và sử dụng `onReadyFirstLayoutRuleRun` để đánh dấu đã sẵn sàng.

**Sai:**

```tsx
// ❌ Render FormBuilder ngay — có thể bị nhấp nháy khi layout rule thay đổi layout
<FormProvider {...methods}>
  <FormBuilder ref={formBuilderRef} items={layout} ... />
</FormProvider>
```

**Đúng:**

```tsx
// ✅ Bọc trong LayoutRuleProvider để kiểm tra trước
const [checkChangeLayoutByLayoutRule, setCheckChangeLayoutByLayoutRule] = useState(false);

<FormProvider {...methods}>
  <LayoutRuleProvider
    layoutType={editMode ? "view" : "create"}
    onlyChangeLayout
    onReadyFirstLayoutRuleRun={() => setCheckChangeLayoutByLayoutRule(true)}
    objectId={objectDetail.id}
    objectSlug={objectDetail.slug}
    layoutSlug=""
    formId=""
  >
    {() => {
      if (!checkChangeLayoutByLayoutRule) return null;
      return <FormBuilder ref={formBuilderRef} items={layout} ... />;
    }}
  </LayoutRuleProvider>
</FormProvider>
```

---

### RULE-FORM-BUILDER-12: Phải kiểm tra dữ liệu sẵn sàng trước khi render FormBuilder

FormBuilder cần `objectDetail`, `layoutResponse`, và `record` (nếu edit mode) đã được load trước khi render. Phải kiểm tra các điều kiện này và return null nếu chưa sẵn sàng.

**Sai:**

```tsx
// ❌ Render FormBuilder khi dữ liệu chưa sẵn sàng — gây lỗi
return (
  <FormProvider {...methods}>
    <FormBuilder items={layout} objectSlug={objectSlug} objectId={objectDetail?.id} ... />
  </FormProvider>
);
```

**Đúng:**

```tsx
// ✅ Kiểm tra dữ liệu trước khi render
if (!layoutResponse || (editMode && !record) || isLoading || !objectDetail) {
  return null;
}

return (
  <FormProvider {...methods}>
    <FormBuilder
      items={layout}
      objectSlug={objectDetail.slug}
      objectId={objectDetail.id}
      ...
    />
  </FormProvider>
);
```

---

### Tổng hợp các hooks liên quan đến FormBuilder

| Hook | Mục đích |
|---|---|
| `useCreateDefaultValues` | Tạo giá trị mặc định từ layout tree và record (nếu có) |
| `useCreateSchema` | Tạo yup schema từ layout tree để validation |
| `useSubmitFormBuilder` | Xử lý toàn bộ lifecycle submit (validate, build body, nested children) |
| `useFormBuilderButton` | Quản lý trạng thái mở/đóng các modal của button group |
| `useHandleClickButtonGroup` | Xử lý sự kiện click button group với các callback tương ứng |
| `useSetValueFieldsByQueryString` | Tự động populate giá trị form từ URL query params |
| `useGetLayout` | Fetch layout data từ API |

### FormBuilderRef (imperative handle)

| Property | Type | Mô tả |
|---|---|---|
| `methods` | `UseFormReturn` | React-hook-form instance |
| `layout` | `FormBuilderLayoutRow[]` | Layout hiện tại |
| `id` | `string` | Form builder ID |
| `objectSlug` | `string` | Object slug |
| `objectId` | `string` | Object ID |
| `formId` | `string` | Form ID |
| `children` | `RefObject<FormBuilderRef[]>` | Ref của các child form (related lists) |
| `editableSubmit` | `(parentRecordId: string) => Promise<void>` | Submit inline edit |
| `createEditableBody` | `() => Record<string, unknown>[]` | Tạo body cho inline edit |
| `handleCreateError` | `(error: object) => void` | Xử lý lỗi khi tạo record |
