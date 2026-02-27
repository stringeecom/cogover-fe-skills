## Cách sử dụng component FormBuilder trong ui-kit

Path: `ui-kit/src/components/FormBuilder`

### Props

| Prop | Bắt buộc | Mô tả |
|---|---|---|
| `items` | ✓ | Layout từ `useGetLayout` |
| `objectSlug` | ✓ | Slug của object |
| `objectId` | ✓ | ID của object |
| `layoutSlug` | ✓ | Slug của layout |
| `ref` | ✓ | `FormBuilderRef` — dùng với `useSubmitFormBuilder` |
| `formId` | — | UUID ổn định qua `useRef(uuid())` |
| `layoutId` | — | ID layout để bật Layout Rules |
| `record` | — | Record hiện tại (edit mode) |
| `defaultValues` | — | Giá trị mặc định (edit mode) |
| `detailForm` | — | `boolean` — `true` cho edit/view mode |
| `layoutType` | — | `"create"` \| `"view"` |
| `canEdit` | — | `boolean` |
| `onEditableSubmit` | — | `async (data) => boolean` — inline save khi blur (detail mode) |
| `disabledLayoutRules` | — | Force-skip layout rules |
| `fieldSlots` | — | `[{ condition: (field) => boolean, render: (field) => ReactNode }]` |
| `groupSlots` | — | `[{ condition: (group) => boolean, render: (group) => ReactNode }]` |

### Hooks liên quan

| Hook | Mục đích |
|---|---|
| `useGetLayout` | Fetch layout từ API |
| `useCreateDefaultValues` | Tạo defaultValues từ layout + record |
| `useCreateSchema` | Tạo yup schema từ layout |
| `useSubmitFormBuilder` | Submit với ref — xử lý nested children |
| `setFormBuilderFieldError` | Set lỗi từ API response |
| `useFormBuilderButton` | Quản lý modal state của button groups |
| `useHandleClickButtonGroup` | Xử lý click button group |

### Basic usage (create mode)

```tsx
const formId = useRef(uuid());
const { layout, isLoading, activeLayoutSlug, activeLayoutId, layoutResponse } = useGetLayout({
  functionLabel: "ADD",
  objectSlug,
  layoutId,
});
const { createDefaultValues } = useCreateDefaultValues();
const { schema } = useCreateSchema({ layout, formId: formId.current });
const values = useMemo(() => createDefaultValues(layout), [createDefaultValues, layout]);
const methods = useForm<Record<string, unknown>>({ mode: "onChange", defaultValues: values, resolver: yupResolver(object(schema)) });
const formBuilderRef = useRef<FormBuilderRef>(null);
const { handleSubmit, isLoading: isSubmitting } = useSubmitFormBuilder({
  ref: formBuilderRef,
  apiUrlSubmit: layoutResponse?.pageSettings?.settingPage?.apiUrlSubmit,
});

if (!layoutResponse || isLoading || !objectDetail) return null;

return (
  <FormProvider {...methods}>
    <FormBuilder ref={formBuilderRef} formId={formId.current}
      items={layout} objectSlug={objectDetail.slug} objectId={objectDetail.id}
      layoutSlug={activeLayoutSlug} layoutId={activeLayoutId} layoutType="create" />
    <Button loading={isSubmitting} onClick={() => handleSubmit()}>Lưu</Button>
  </FormProvider>
);
```

### Edit/detail mode

```tsx
<FormBuilder
  ref={formBuilderRef} formId={formId.current}
  items={layout} objectSlug={objectDetail.slug} objectId={objectDetail.id}
  layoutSlug={activeLayoutSlug} layoutId={activeLayoutId}
  layoutType="view" detailForm={true} record={record} defaultValues={values}
  onEditableSubmit={async (data) => {
    try { await updateRecord({ objectId: objectDetail.id, id: record.id, data }); return true; }
    catch { return false; }
  }}
/>
```

### API error handling

```tsx
import { setFormBuilderFieldError } from "@stringeecom/ui-kit";
const { mutateAsync } = useUpdateRecord({
  onError: (error) => {
    const errorMeta = objectPath.get(error, "response.data.body.meta", {});
    setFormBuilderFieldError({ formId: formId.current, errors: errorMeta });
  },
});
```

### DO / DON'T

DO: Bọc FormBuilder trong `FormProvider` — component đọc form state từ context.

DO: Tạo `formId` bằng `useRef(uuid())` — không tạo inline `uuid()` (thay đổi mỗi render).

DO: Kiểm tra `!layoutResponse || (editMode && !record) || isLoading || !objectDetail` trước khi render.

DO: Dùng `fieldSlots`/`groupSlots` để custom render thay vì override CSS `[&_.form-builder-group]`.

DON'T: Tự tạo defaultValues thủ công — dùng `useCreateDefaultValues`.

DON'T: Tự gọi `methods.handleSubmit` — dùng `useSubmitFormBuilder` với ref.

DON'T: Truyền `layoutId=""` — không truyền khi không có layout rules.
