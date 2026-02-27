---
name: cogover-fe
description: Skill tổng hợp cho dự án Cogover Frontend. Bao gồm 3 phần chính - (1) UI Kit với foundations (design tokens, màu sắc, typography, spacing, cx utility), common components, form components, và hooks; (2) Create Commit - tạo git commit tuân theo quy ước commit message của Cogover; (3) i18n - dịch text tiếng Việt sang đa ngôn ngữ. Sử dụng khi viết/review code UI, tạo commit, hoặc dịch i18n.
license: MIT
metadata:
  author: cogover
  version: "3.0.0"
---

# UI Kit

Quy tắc sử dụng ui-kit trong dự án Cogover Frontend. Bao gồm foundations, common components, form components, và hooks.

## Khi nào áp dụng

Tham khảo các hướng dẫn này khi:

### Import

- Import bất kỳ component, hook, hoặc utility nào từ ui-kit — luôn import từ `@stringeecom/ui-kit`

### Form

- Tạo form với react-hook-form, FormProvider, Controlled components, yup validation — xem chi tiết tại `uikit-form-react-hook-form`

### Object, Field, Record

- Mô hình dữ liệu Object-Field-Record, các hook lấy/tạo/sửa/xóa record, kiểm tra quyền xem/sửa field — xem chi tiết tại `uikit-object-field-record`

### Text & i18n

- Tất cả text mới tạo ra phải viết bằng tiếng Việt, chỉ dịch sang i18n key khi được yêu cầu sử dụng skill `ui-i18n`

### Date/Time

- Làm việc với ngày giờ: parse, format, so sánh, tính toán, chuyển đổi timestamp — luôn dùng `dayjs`, xem chi tiết tại `dayjs-date-time`

### Page Patterns

> Không cần tham khảo cấu trúc của các list page khác trong dự án, chỉ cần tuân theo các quy tắc trong rule `page-list-resource`.
- `RULE-LIST-PAGE-01` — Cấu trúc thư mục: `index.tsx`, `type.ts`, `options.tsx`, `components/{Resource}List/`, `FilterMenu/`, `ActionMenu/`
- `RULE-LIST-PAGE-02` — Tầng Type: enum FilterType, interface FilterForm với operator đi kèm mỗi filter field
- `RULE-LIST-PAGE-03` — Options tách riêng file, label luôn là i18n key
- `RULE-LIST-PAGE-04` — Trang chính: FormProvider + useGetStateFromSearchParams + SyncFilterFormWithSearchParams + ResizableMenu
- `RULE-LIST-PAGE-05` — Bảng dữ liệu: TableSettings + ListViewTable + useTableV2 + useDebounceValue + buildParamsFromFilter
- `RULE-LIST-PAGE-06` — Helper: guard bằng `filterTypes.includes()`, chỉ gửi param có giá trị, normalize date range
- `RULE-LIST-PAGE-07` — useGetColumns hook: useMemo, nhận action callbacks, dùng SmartTooltip/CreatedByInfo
- `RULE-LIST-PAGE-08` — FilterMenu: useFormContext, render có điều kiện theo filterTypes, sync URL, appendToBody={false}
- `RULE-LIST-PAGE-09` — ActionMenu: Popper + PopperMenu + PopperMenuItem, spread `{...params} {...attributes}`, hide-on-click
- `RULE-LIST-PAGE-10` — Flow: URL → Form → useWatch → debounce → buildParams → React Query → Table

### Workflow

- Trước khi làm, phân tích yêu cầu để tách công việc độc lập — nếu tách được từ 2 công việc trở lên thì tự động dùng sub agent song song
- Không bao giờ tự tạo git commit trừ khi được user yêu cầu rõ ràng

### Foundations

- Cấu hình Tailwind CSS hoặc design tokens
- Làm việc với color palette, spacing, hoặc typography scales
- Thiết lập CSS custom properties / variables
- Quản lý themes (light/dark mode)

### Common Components

- Sử dụng `Alert` từ ui-kit cho hiển thị thông báo warning/info
- Sử dụng `Avatar` hoặc `AvatarField` từ ui-kit cho hiển thị ảnh đại diện
- Sử dụng `BreadCrumbs` từ ui-kit cho navigation breadcrumbs với dropdown
- Sử dụng `Button`, `ButtonGroup`, `IconButton`, `Tooltip`, hoặc `SmartTooltip` từ ui-kit
- Sử dụng `Checkbox`, `CheckboxGroup`, hoặc `FormControlLabel` từ ui-kit
- Sử dụng `CKEditor` từ ui-kit cho rich text editor
- Sử dụng `CollapsibleContainer` từ ui-kit
- Sử dụng `ConfirmModal` cho xác nhận hành động nguy hiểm
- Sử dụng `CreatedByInfo` để hiển thị thông tin creator (avatar + name)
- Sử dụng `DatePicker` hoặc `DatePickerRelative` từ ui-kit cho chọn ngày/giờ
- Sử dụng `DateRangeCalendar` từ ui-kit cho chọn khoảng ngày trên calendar
- Sử dụng `DateRangePicker` từ ui-kit cho chọn khoảng ngày với input
- Sử dụng `DraggableWrapper` và `DraggableItem` từ ui-kit cho sortable drag-and-drop (react-beautiful-dnd)
- Sử dụng `Drawer` từ ui-kit cho side panel/drawer
- Sử dụng `FilePreview` hoặc `BaseFilePreview` từ ui-kit cho xem trước file
- Sử dụng `FilterGenerator` từ ui-kit cho bộ lọc điều kiện với DataField
- Sử dụng `FilterGeneratorBase` từ ui-kit cho bộ lọc điều kiện tuỳ chỉnh (base component)
- Sử dụng `FormBuilder` từ ui-kit cho form động dựa trên layout từ server (server-driven form)
- Sử dụng `FormControlLabel` từ ui-kit để gắn label cho control (Checkbox, Radio, Switch)
- Sử dụng `HelpButton` để thêm inline contextual help
- Sử dụng `HorizontalSteps` để implement multi-step wizard UI
- Sử dụng `InsertFieldObjectButtonV2` từ ui-kit cho nút mở modal chọn trường dữ liệu
- Sử dụng `InsertFieldObjectButtonWrapper2` từ ui-kit cho wrapper hiển thị biến đã chọn với nút clear
- Sử dụng `InsertFieldObjectV2` từ ui-kit cho modal chọn trường dữ liệu dạng cascader
- Sử dụng `LayoutSelectButton` từ ui-kit cho nút chọn layout hiển thị
- Sử dụng `LimitableList` từ ui-kit cho hiển thị danh sách với giới hạn số lượng
- Sử dụng `ListViewTable` từ ui-kit cho bảng dữ liệu có header với action buttons
- Sử dụng `Modal` và `ModalFooter` để xây dựng dialog và overlay
- Sử dụng `Popper`, `PopperMenu`, hoặc `PopperMenuItem` từ ui-kit
- Sử dụng `ResizeBar` để implement resizable panels hoặc sidebar
- Sử dụng `Skeleton` từ ui-kit cho loading placeholder
- Sử dụng `StepProgressBar` để hiển thị step-based progress
- Sử dụng `Steps` để render step wizard indicator
- Sử dụng `StringeeDndContext`, `DndContainer`, và `useDndHandle` cho drag-and-drop
- Sử dụng `Table` từ ui-kit cho hiển thị bảng dữ liệu
- Sử dụng `TableSettings` từ ui-kit cho cài đặt cột bảng (ẩn/hiện, ghim, kéo thả, editable)
- Sử dụng `Tabs` để xây dựng tab navigation
- Sử dụng `Tag` để hiển thị labels, chips, hoặc removable tags
- Sử dụng `showToastMessage` cho in-app notifications
- Sử dụng `UsageHelpButton` để hiển thị contextual feature documentation
- Sử dụng `VerticalSteps` để render vertical sidebar step navigator
- Sử dụng hook `useForceUpdate` cho force re-render component
- Sử dụng hook `useGetDataFieldBySlug` cho lấy data field theo slug/ID từ object
- Sử dụng hook `useGetDataFieldOptionLabel` cho lấy label option của data field
- Sử dụng hook `useGetFieldLabel` cho lấy label hiển thị của field từ slug
- Sử dụng hook `useGetLayout` cho quản lý form layout (standard/custom)
- Sử dụng hook `useGetRecordPageLink` cho tạo link điều hướng record (serial/normal)
- Sử dụng hook `useGetStateFromSearchParams` cho lấy typed state từ URL search params
- Sử dụng hook `useHistoriesState` cho undo/redo state management
- Sử dụng hook `useMergeRecordFilters` cho merge nhiều bộ lọc record
- Sử dụng hook `useMergedRefs` cho merge nhiều refs lại với nhau
- Sử dụng hook `useReplaceRecordValue` cho thay thế template variables bằng giá trị record
- Sử dụng hook `useSaveToSearchParams` cho lưu state vào URL search params
- Sử dụng hook `useSyncState` cho đồng bộ state giữa các tab trình duyệt
- Review code có sử dụng bất kỳ component nào ở trên
- Lựa chọn giữa `Tooltip` vs `SmartTooltip`, hoặc `Button onlyIcon` vs `IconButton`
- Lựa chọn giữa controlled vs uncontrolled mode cho `CollapsibleContainer`

### Form Components

- Sử dụng `DurationInput` hoặc `DurationRangeInput` từ ui-kit cho nhập thời lượng
- Sử dụng `EmailInput` từ ui-kit cho input email với nút mailto
- Sử dụng `EmojiControlPicker` hoặc `EmojiPicker` từ ui-kit cho chọn emoji
- Sử dụng `FileInput` từ ui-kit cho upload file với drag-and-drop
- Sử dụng `FormItem` từ ui-kit cho layout label + component trong form
- Sử dụng `InputGroup` từ ui-kit cho input kết hợp select
- Sử dụng `InputNumber` từ ui-kit cho nhập số với format
- Sử dụng `MarkdownEditor` từ ui-kit cho markdown editor
- Sử dụng `MonacoEditor` từ ui-kit cho code editor
- Sử dụng `MultipleSelect` từ ui-kit cho chọn nhiều giá trị
- Sử dụng `PhoneInput` từ ui-kit cho nhập số điện thoại với mã quốc gia
- Sử dụng `Radio`, `RadioGroup` từ ui-kit cho radio buttons
- Sử dụng `Rating` từ ui-kit cho đánh giá sao
- Sử dụng `Reaction` từ ui-kit cho chọn emoji reaction
- Sử dụng `SearchInput` từ ui-kit cho input tìm kiếm
- Sử dụng `Select` từ ui-kit cho dropdown select
- Sử dụng `Slider` từ ui-kit cho range/volume inputs
- Sử dụng `Switch` từ ui-kit cho toggle bật/tắt
- Sử dụng `TagsInput` từ ui-kit cho nhập tags
- Sử dụng `TextField` từ ui-kit cho text input/textarea
- Sử dụng `TextLink` từ ui-kit cho hyperlink text
- Sử dụng `TimePicker` từ ui-kit cho chọn giờ
- Sử dụng `TimeRangePicker` từ ui-kit cho chọn khoảng thời gian
- Sử dụng `TimeSelect` từ ui-kit cho danh sách chọn giờ dạng scroll
- Sử dụng `TreeSelect` từ ui-kit cho chọn từ cây phân cấp
- Sử dụng `UrlInput` từ ui-kit cho input URL với nút mở link
- Lựa chọn giữa controlled vs uncontrolled mode cho form inputs

## Tham khảo nhanh


### Foundations

#### 1. Design Tokens (QUAN TRỌNG)

- `tokens-tailwind-config` - Cách định nghĩa và sử dụng design tokens qua tailwind.config.js

#### 2. Hệ thống màu sắc (QUAN TRỌNG)

- `color-palette-usage` - Luôn sử dụng configured color classes, không bao giờ dùng raw hex/rgb hoặc default Tailwind palette

#### 3. Typography (QUAN TRỌNG)

- `typography-prose-classes` - Sử dụng `prose-<type>` cho font-size, font-weight, line-height; đọc types từ tailwind.config.js; default body text = `prose-body2`

#### 4. Spacing & Layout (QUAN TRỌNG)

- `spacing-rem-only` - Luôn sử dụng rem, không bao giờ dùng px (1rem = 16px); ngoại lệ duy nhất là 1px borders

#### 5. className Composition (CAO)

- `cx-class-grouping` - Luôn import và sử dụng `cx()`; nhóm các class liên quan vào các argument riêng biệt; sử dụng object syntax cho conditionals

#### 6. TypeScript Type Safety (QUAN TRỌNG)

- `no-typescript-any` - Không bao giờ sử dụng kiểu `any` và dấu `!` (non-null assertion); sử dụng `unknown`, `Record<string, unknown>`, generic types, optional chaining thay thế

#### 7. Prettier & Lint Check (CAO)

- `prettier-and-lint-check` - Sau khi sửa code xong phải kiểm tra `.prettierrc` ở root và chạy prettier, sau đó kiểm tra và sửa hết lỗi lint trong các file vừa sửa

#### 8. Không Tự Tạo File Markdown (CAO)

- `no-auto-markdown` - Không tự động tạo file markdown trừ khi user yêu cầu rõ ràng

#### 9. Date/Time với dayjs (CAO)

- `dayjs-date-time` - Luôn sử dụng dayjs cho mọi thao tác date/time; không dùng native Date, moment.js, hoặc tự tính toán

### Common Components

#### Alert (`uikit-alert`)

- `RULE-ALERT-01` — Sử dụng `variant` phù hợp với ngữ cảnh thông báo — `"warning"` hoặc `"info"`
- `RULE-ALERT-02` — Không truyền rõ ràng giá trị mặc định cho `variant` và `icon`
- `RULE-ALERT-03` — Truyền `icon={null}` để ẩn icon, không truyền icon rỗng
- `RULE-ALERT-04` — Sử dụng `title` khi cần phân biệt tiêu đề và nội dung chi tiết
- `RULE-ALERT-05` — Sử dụng `messageClassName` và `titleClassName` để tuỳ chỉnh style

#### Avatar (`uikit-avatar`)

- `RULE-AVT-01` — Luôn truyền `alt` để hiển thị tên viết tắt khi không có ảnh
- `RULE-AVT-02` — Sử dụng `fallbackImage` thay vì tự xử lý onError
- `RULE-AVT-03` — Sử dụng `size` (số, đơn vị px) thay vì className để đặt kích thước
- `RULE-AVT-04` — Sử dụng prop `badge` để hiển thị badge, không tự wrap thêm Avatar
- `RULE-AVT-05` — Truyền `id` để kích hoạt popup thông tin nhân sự và màu nền tự động
- `RULE-AVT-06` — Sử dụng `personnelDetailPopperProps` để tùy chỉnh popup, không tự wrap Popper
- `RULE-AVT-07` — Không tự tạo component avatar với img tag hoặc div

#### AvatarField (`uikit-avatar-field`)

- `RULE-AVATARFIELD-01` — Sử dụng `AvatarField` thay vì tự xử lý URL từ `ImageFieldResponse`
- `RULE-AVATARFIELD-02` — Chọn `resolutionSize` phù hợp với kích thước hiển thị
- `RULE-AVATARFIELD-03` — `AvatarField` chấp nhận cả URL string lẫn `ImageFieldResponse`
- `RULE-AVATARFIELD-04` — Sử dụng `ImageField` khi cần hiển thị hình ảnh (không phải avatar)
- `RULE-AVATARFIELD-05` — `AvatarField` kế thừa tất cả props của `Avatar`
- `RULE-AVATARFIELD-06` — Truyền `alt` cho avatar để đảm bảo accessibility

#### BreadCrumbs (`uikit-breadcrumbs`)

- `RULE-BREADCRUMBS-01` — Truyền `items` đúng cấu trúc `BreadCrumbItems[]` — mỗi item phải có `label`
- `RULE-BREADCRUMBS-02` — Sử dụng `children` để tạo dropdown menu cho item
- `RULE-BREADCRUMBS-03` — Sử dụng `isAutoRedirect` khi item có `children` và cần click để điều hướng
- `RULE-BREADCRUMBS-04` — Sử dụng `isFullLabel` khi không muốn label bị cắt ngắn
- `RULE-BREADCRUMBS-05` — Sử dụng `isDisabled` để vô hiệu hóa điều hướng
- `RULE-BREADCRUMBS-06` — Sử dụng `isShow` trên item để ẩn item, không dùng filter bên ngoài

#### Button (`uikit-button`)

- `RULE-BTN-01` — Sử dụng `Button onlyIcon` (không phải bare `IconButton`) khi cần tooltip
- `RULE-BTN-02` — Sử dụng prop `loading` thay vì manual spinner + disabled
- `RULE-BTN-03` — Sử dụng `startIcon`/`endIcon`, không phải icon as children
- `RULE-BTN-04` — Sử dụng `disabledTooltip` để suppress tooltip trên `onlyIcon` buttons

#### ButtonGroup (`uikit-button-group`)

- `RULE-BTN-GROUP-01` — Sử dụng `ButtonGroup` để nhóm các button liên quan, không dùng manual flex wrappers
- `RULE-BTN-GROUP-02` — Set `size` trên `ButtonGroup`, không phải trên từng children
- `RULE-BTN-GROUP-03` — Sử dụng `row={false}` cho vertical layout (mặc định là horizontal)
- `RULE-BTN-GROUP-04` — Không nest `ButtonGroup` bên trong `ButtonGroup` khác

#### Checkbox & CheckboxGroup (`uikit-checkbox`)

- `RULE-CHECKBOX-01` — Pair `checked` với `onChange` trong controlled mode — omit `checked` cho uncontrolled
- `RULE-CHECKBOX-02` — Sử dụng `indeterminate` cho trạng thái "chọn một phần" — không tự build icon
- `RULE-CHECKBOX-03` — Sử dụng `size` cho kích thước — không override width/height qua className
- `RULE-CHECKBOX-04` — Không truyền `type` — component luôn là checkbox
- `RULE-CHECKBOX-05` — Sử dụng `CheckboxGroup` để quản lý nhóm checkbox — không tự viết logic toggle mảng
- `RULE-CHECKBOX-06` — Sử dụng `FormControlLabel` để thêm label cho checkbox trong CheckboxGroup
- `RULE-CHECKBOX-07` — Sử dụng `row` trên CheckboxGroup cho layout ngang — không override flex direction
- `RULE-CHECKBOX-08` — `disabled` trên CheckboxGroup override disabled của từng children
- `RULE-CHECKBOX-09` — Mỗi Checkbox trong CheckboxGroup phải có `value` unique
- `RULE-CHECKBOX-10` — Sử dụng `labelPlacement` trên FormControlLabel để đổi vị trí label — không đảo thứ tự JSX

#### CheckboxGroup (`uikit-checkbox-group`)

- `RULE-CBXGRP-01` — Luôn truyền `value` (mảng) và `onChange` — CheckboxGroup là controlled component
- `RULE-CBXGRP-02` — Không tự viết logic toggle mảng — CheckboxGroup đã xử lý sẵn
- `RULE-CBXGRP-03` — Mỗi Checkbox con phải có `value` unique
- `RULE-CBXGRP-04` — Sử dụng `row` để chuyển layout ngang — không override flex direction qua className
- `RULE-CBXGRP-05` — `disabled` trên CheckboxGroup override disabled của từng children
- `RULE-CBXGRP-06` — Sử dụng `FormControlLabel` để thêm label — không tự tạo layout label + checkbox
- `RULE-CBXGRP-07` — Không tự wrap container div — CheckboxGroup đã render div với flex layout
- `RULE-CBXGRP-08` — Generic type T được suy luận từ `value` — đảm bảo kiểu nhất quán

#### CKEditor (`uikit-ckeditor`)

- `RULE-CKEDITOR-01` — Luôn truyền `value` và `onChange` — CKEditor là controlled component
- `RULE-CKEDITOR-02` — Sử dụng `minHeight`/`maxHeight` hoặc `autoGrow` cho editor co giãn — không override CSS height
- `RULE-CKEDITOR-03` — Truyền đủ `uploadImageUrl` + `getUrlFromFileUploadResponse` để upload ảnh hoạt động
- `RULE-CKEDITOR-04` — Cấu hình `autocompleteConfigs` đầy đủ `pattern` và `dataCallback` — không tự build autocomplete
- `RULE-CKEDITOR-05` — Sử dụng ref `setTextValue` để chèn text — không truy cập DOM trực tiếp
- `RULE-CKEDITOR-06` — Sử dụng `error` và `helperText` cho validation — không tự thêm helper text bên ngoài
- `RULE-CKEDITOR-07` — Sử dụng `stickyTopToolbar` khi editor nằm trong scrollable container
- `RULE-CKEDITOR-08` — Sử dụng `disabledModules` để tắt toolbar items — không ẩn bằng CSS

#### CollapsibleContainer (`uikit-collapsible-container`)

- `RULE-COLLAPSIBLE-01` — Sử dụng uncontrolled mode khi không cần external state
- `RULE-COLLAPSIBLE-02` — Luôn pair `collapsed` với `onToggleCollapse` trong controlled mode
- `RULE-COLLAPSIBLE-03` — Sử dụng `unmountOnHide={false}` để preserve children state
- `RULE-COLLAPSIBLE-04` — Không manually control visibility của children
- `RULE-COLLAPSIBLE-05` — Pass `title` as `ReactNode` cho rich header content

#### ConfirmModal (`uikit-confirm-modal`)

- `RULE-CONFIRM-01` — Sử dụng `ConfirmModal` cho tất cả destructive action confirmations, không bao giờ build custom
- `RULE-CONFIRM-02` — Omit `onCancel` khi nó có behavior giống `onClose`
- `RULE-CONFIRM-03` — Sử dụng prop `loading` thay vì manual disabled + spinner
- `RULE-CONFIRM-04` — Pass `message` as `ReactNode` cho rich content
- `RULE-CONFIRM-05` — Customize `cancelText`/`confirmText` khi default text không rõ ràng

#### CopyLinkButton (`uikit-copy-link-button`)

- `RULE-COPYLINK-01` — Sử dụng `CopyLinkButton` thay vì tự xây dựng logic sao chép URL
- `RULE-COPYLINK-02` — Truyền đường dẫn tương đối cho prop `link`, không truyền full URL
- `RULE-COPYLINK-03` — Không tự quản lý trạng thái copied bên ngoài component
#### CreatedByInfo (`uikit-created-by-info`)

- `RULE-CREATED-BY-01` — Sử dụng `isSystem` cho Automation Cogover, không phải manual markup
- `RULE-CREATED-BY-02` — Luôn pass `id` để đảm bảo avatar fallback color nhất quán
- `RULE-CREATED-BY-03` — Pass object `ImageResponse` trực tiếp khi avatar đến từ API
- `RULE-CREATED-BY-04` — Không wrap với custom tooltip — SmartTooltip đã built-in
- `RULE-CREATED-BY-05` — Không recreate layout này manually với Avatar + name span

#### DatePicker (`uikit-date-picker`)

- `RULE-DATE-PICKER-01` — `value` là `Dayjs | null` — không truyền Date, string, hay timestamp
- `RULE-DATE-PICKER-02` — Format có chứa giờ tự kích hoạt DateTime mode — không cần prop riêng
- `RULE-DATE-PICKER-03` — Sử dụng `min`/`max` là `Dayjs` để giới hạn khoảng ngày — không tự validate bên ngoài
- `RULE-DATE-PICKER-04` — Sử dụng `clearable` để cho phép xoá giá trị — mặc định đã bật
- `RULE-DATE-PICKER-05` — Sử dụng `useTimeAgoValue` với `timeAgoValue` + `onChangeTimeAgoValue` cho preset time ago
- `RULE-DATE-PICKER-06` — Sử dụng `renderValue` để tuỳ chỉnh hiển thị — không override input value bên ngoài
- `RULE-DATE-PICKER-07` — Sử dụng `calendarProps` để truyền config xuống Calendar — không tự build calendar
- `RULE-DATE-PICKER-08` — Input hỗ trợ nhập tay — blur sẽ parse và validate theo `format`

#### DateRangeCalendar (`uikit-date-range-calendar`)

- `RULE-DATE-RANGE-CAL-01` — `value` là tuple `[Dayjs | null, Dayjs | null]` — không truyền Date, string
- `RULE-DATE-RANGE-CAL-02` — `onChange` trả về một `Dayjs` duy nhất — tự quản lý logic from/to bên ngoài
- `RULE-DATE-RANGE-CAL-03` — Sử dụng `focusedValueIndex` để điều khiển focus vào from (0) hoặc to (1)
- `RULE-DATE-RANGE-CAL-04` — Bật `dateTime` để hiển thị TimeSelect — không tự build time picker
- `RULE-DATE-RANGE-CAL-05` — Sử dụng `min`/`max` là `Dayjs` để giới hạn khoảng ngày
- `RULE-DATE-RANGE-CAL-06` — Sử dụng `disabledKeyboard` khi dùng trong Popper/Popup
- `RULE-DATE-RANGE-CAL-07` — Sử dụng `headerTextFormat` để tuỳ chỉnh header — không tự build
- `RULE-DATE-RANGE-CAL-08` — Highlight khoảng ngày được xử lý tự động — không cần tuỳ chỉnh

#### DateRangePicker (`uikit-date-range-picker`)

- `RULE-DATE-RANGE-PICKER-01` — `value` là `[Dayjs | null, Dayjs | null]` — không truyền Date, string
- `RULE-DATE-RANGE-PICKER-02` — Format có chứa giờ tự kích hoạt DateTime mode — không cần prop riêng
- `RULE-DATE-RANGE-PICKER-03` — Sử dụng `fromMin`/`fromMax`/`toMin`/`toMax` để giới hạn khoảng ngày
- `RULE-DATE-RANGE-PICKER-04` — Component tự động hoán đổi from/to khi from > to
- `RULE-DATE-RANGE-PICKER-05` — Sử dụng `renderValue` để tuỳ chỉnh hiển thị
- `RULE-DATE-RANGE-PICKER-06` — Sử dụng `calendarProps` để truyền config xuống DateRangeCalendar
- `RULE-DATE-RANGE-PICKER-07` — Input hỗ trợ nhập tay — blur sẽ parse và validate theo `format`
- `RULE-DATE-RANGE-PICKER-08` — Sử dụng `helperText` và `error` cho thông báo lỗi
- `RULE-DATE-RANGE-PICKER-09` — Sử dụng `getFromRef` và `getToRef` để truy cập input ref
- `RULE-DATE-RANGE-PICKER-10` — Xoá giá trị bằng icon clear có sẵn — không tự build nút clear

#### DraggableWrapper & DraggableItem (`uikit-draggable-wrapper`, `uikit-draggable-item`)

- `RULE-DW-01` — Luôn sử dụng `DraggableWrapper` kết hợp với `DraggableItem` — không dùng trực tiếp `react-beautiful-dnd`
- `RULE-DW-02` — `children` của `DraggableItem` phải là render function nhận `dragHandleProps`
- `RULE-DW-03` — Luôn spread `dragHandleProps` lên phần tử dùng làm tay cầm kéo
- `RULE-DW-04` — `draggableId` phải là duy nhất — không sử dụng `index`
- `RULE-DW-05` — Sử dụng `direction="horizontal"` khi cần kéo thả ngang — mặc định `"vertical"`
- `RULE-DW-06` — Xử lý `destinationIndex` có thể là `undefined` trong `onDragEnd`
- `RULE-DW-07` — Truyền `hasDraggable={true}` vào `Modal` khi DraggableWrapper bên trong
- `RULE-DW-08` — Sử dụng `isDragDisabled` để vô hiệu hóa kéo — không ẩn `dragHandleProps`
- `RULE-DI-01` — Luôn sử dụng `DraggableItem` bên trong `DraggableWrapper`
- `RULE-DI-02` — `children` là render function — phải spread `dragHandleProps` lên drag handle
- `RULE-DI-03` — Spread `dragHandleProps` lên phần tử con cụ thể khi cần drag handle riêng
- `RULE-DI-04` — `draggableId` phải là duy nhất và chuỗi ổn định
- `RULE-DI-05` — `index` phải là số thứ tự liên tục bắt đầu từ 0
- `RULE-DI-06` — Sử dụng `isDragDisabled` để vô hiệu hóa kéo có điều kiện
- `RULE-DI-07` — Khi sử dụng trong `Modal`, phải truyền `hasDraggable={true}`

#### Drawer (`uikit-drawer`)

- `RULE-DRAWER-01` — Render có điều kiện Drawer để mở/đóng — không có prop `open`/`visible`
- `RULE-DRAWER-02` — Sử dụng `placement` để xác định vị trí hiển thị
- `RULE-DRAWER-03` — Sử dụng `width` và `height` để tùy chỉnh kích thước
- `RULE-DRAWER-04` — Truyền `onOverlayClick` để đóng drawer khi nhấn overlay
- `RULE-DRAWER-05` — Sử dụng `hasOverlay={false}` khi không cần overlay
- `RULE-DRAWER-06` — Không truyền rõ ràng các giá trị mặc định
- `RULE-DRAWER-07` — Sử dụng `title` để hiển thị header với nút đóng
- `RULE-DRAWER-08` — Quản lý `triggerEscPress` khi có modal/dialog lồng nhau
- `RULE-DRAWER-09` — Sử dụng `headingClassName` và `overlayClassName` để tùy chỉnh style

#### FilterGenerator (`uikit-filter-generator`)

- `RULE-FILTER-GENERATOR-01` — FilterGenerator là controlled component — phải truyền đầy đủ `logicType`, `logicValue`, `filterItems` và các hàm onChange tương ứng
- `RULE-FILTER-GENERATOR-02` — `dataFields` phải là mảng `DataField[]` hợp lệ — không truyền mảng rỗng hoặc dữ liệu không đúng kiểu
- `RULE-FILTER-GENERATOR-03` — Sử dụng `logicType` để chọn kiểu logic kết hợp — không tự build logic select bên ngoài
- `RULE-FILTER-GENERATOR-04` — Sử dụng `logicType="CUSTOM"` kết hợp `logicValue` để tạo biểu thức logic tuỳ chỉnh
- `RULE-FILTER-GENERATOR-05` — Sử dụng `required` để ẩn tuỳ chọn "NONE" trong logic select
- `RULE-FILTER-GENERATOR-06` — Sử dụng `recordDataFields` kết hợp `text` để hiển thị trường dữ liệu record theo nhóm
- `RULE-FILTER-GENERATOR-07` — Sử dụng `clearable` kết hợp `onReset` để cho phép reset filter
- `RULE-FILTER-GENERATOR-08` — `disabled` và `readOnly` có hành vi khác nhau — không nhầm lẫn
- `RULE-FILTER-GENERATOR-09` — Sử dụng `maxFilterItemCount` để giới hạn số điều kiện
- `RULE-FILTER-GENERATOR-10` — Sử dụng `showPreview` và `onClickPreview` để hiển thị nút xem trước
- `RULE-FILTER-GENERATOR-11` — Sử dụng `insertFieldObjectButtonWrapperProps` khi cần chọn giá trị từ biến
- `RULE-FILTER-GENERATOR-12` — Sử dụng `renderValueInputWrapper` để tuỳ chỉnh wrapper của input giá trị
- `RULE-FILTER-GENERATOR-13` — Sử dụng `showCustomLogicError` để kiểm soát hiển thị lỗi logic tuỳ chỉnh
- `RULE-FILTER-GENERATOR-14` — Sử dụng `FilterGeneratorItem` đúng cấu trúc khi khởi tạo filterItems
- `RULE-FILTER-GENERATOR-15` — Sử dụng `onChangeFilterItemParams` khi cần theo dõi thay đổi params của từng điều kiện

#### FilterGeneratorBase (`uikit-filter-generator-base`)

- `RULE-FILTER-GENERATOR-BASE-01` — FilterGeneratorBase là controlled component — phải truyền đầy đủ state và callback
- `RULE-FILTER-GENERATOR-BASE-02` — Phải khai báo generic type cho `FilterGeneratorBase` và `FilterGeneratorBaseItem`
- `RULE-FILTER-GENERATOR-BASE-03` — Sử dụng `renderField` để render field selector — không tự build bên ngoài
- `RULE-FILTER-GENERATOR-BASE-04` — Sử dụng `renderOperators` để render operator dựa trên field đã chọn
- `RULE-FILTER-GENERATOR-BASE-05` — Sử dụng `renderValueInput` để render input giá trị dựa trên field và operator
- `RULE-FILTER-GENERATOR-BASE-06` — Sử dụng `logicType` để chọn kiểu kết hợp điều kiện
- `RULE-FILTER-GENERATOR-BASE-07` — Sử dụng `minFilterItemCount` để giới hạn số điều kiện tối thiểu
- `RULE-FILTER-GENERATOR-BASE-08` — Sử dụng `maxFilterItemCount` để giới hạn số điều kiện tối đa
- `RULE-FILTER-GENERATOR-BASE-09` — Sử dụng `getNewParamsWhenChangeOperator` để reset params khi đổi operator
- `RULE-FILTER-GENERATOR-BASE-10` — Sử dụng `getNewOperatorWhenChangeField` để tự động đổi operator khi đổi field
- `RULE-FILTER-GENERATOR-BASE-11` — Sử dụng `text` để tuỳ chỉnh nhãn cột
- `RULE-FILTER-GENERATOR-BASE-12` — Sử dụng `required` để ẩn lựa chọn "NONE" trong logic type
- `RULE-FILTER-GENERATOR-BASE-13` — Sử dụng `disabled` để vô hiệu hoá toàn bộ bộ lọc
- `RULE-FILTER-GENERATOR-BASE-14` — Sử dụng `showCustomLogicError` để kiểm soát hiển thị lỗi custom logic

#### FormBuilder (`uikit-form-builder`)

- `RULE-FORM-BUILDER-01` — FormBuilder bắt buộc phải nằm trong FormProvider của react-hook-form
- `RULE-FORM-BUILDER-02` — Phải sử dụng useCreateDefaultValues và useCreateSchema để tạo defaultValues và schema từ layout
- `RULE-FORM-BUILDER-03` — Phải truyền ref cho FormBuilder và sử dụng useSubmitFormBuilder để submit
- `RULE-FORM-BUILDER-04` — Sử dụng formId từ uuid, không tự tạo string bất kỳ
- `RULE-FORM-BUILDER-05` — Phân biệt rõ layoutType và detailForm cho create mode và edit mode
- `RULE-FORM-BUILDER-06` — Truyền layoutId để bật layout rules — không truyền khi không cần
- `RULE-FORM-BUILDER-07` — Sử dụng useGetLayout để lấy layout data — không tự build layout tree
- `RULE-FORM-BUILDER-08` — Sử dụng setFormBuilderFieldError để hiển thị lỗi từ API response
- `RULE-FORM-BUILDER-09` — Sử dụng useFormBuilderButton và useHandleClickButtonGroup để xử lý button group
- `RULE-FORM-BUILDER-10` — Sử dụng groupSlots và fieldSlots để custom render — không override CSS nội bộ
- `RULE-FORM-BUILDER-11` — Sử dụng LayoutRuleProvider khi cần kiểm tra change layout trước khi render FormBuilder
- `RULE-FORM-BUILDER-12` — Phải kiểm tra dữ liệu sẵn sàng trước khi render FormBuilder

#### FilePreview (`uikit-file-preview`)

- `RULE-FP-01` — Chọn đúng variant `FilePreview` hoặc `BaseFilePreview`
- `RULE-FP-02` — Render có điều kiện để mở/đóng FilePreview
- `RULE-FP-03` — Sử dụng controlled `activeIndex` khi cần đồng bộ trạng thái bên ngoài
- `RULE-FP-04` — Không truyền rõ ràng các giá trị mặc định cho `scaleStep`, `minScale`, `maxScale`
- `RULE-FP-05` — Sử dụng `getPublicUrl` trong `BaseFilePreview` cho file cần xác thực
- `RULE-FP-06` — Không tự xây dựng nút điều hướng hoặc zoom cho FilePreview
- `RULE-FP-07` — Truyền mảng `files` chứa ít nhất một phần tử

#### FormControlLabel (`uikit-form-control-label`)

- `RULE-FCL-01` — Sử dụng `FormControlLabel` để gắn label cho control — không tự tạo layout
- `RULE-FCL-02` — Truyền component vào prop `control` — không truyền vào children
- `RULE-FCL-03` — Sử dụng `labelPlacement` để đổi vị trí label — không đảo thứ tự JSX
- `RULE-FCL-04` — Sử dụng `labelAlign` để căn chỉnh label
- `RULE-FCL-05` — Truyền `disabled` trên FormControlLabel để đồng bộ disabled state
- `RULE-FCL-06` — Sử dụng `disabledLabel` để kiểm soát style label khi disabled
- `RULE-FCL-07` — Sử dụng `usingHtmlFor={false}` khi control không hỗ trợ htmlFor
- `RULE-FCL-08` — Sử dụng `classNameLabel` để style label — không override qua `className`
- `RULE-FCL-09` — Label hỗ trợ ReactNode — sử dụng cho nội dung phức tạp

#### HelpButton (`uikit-help-button`)

- `RULE-HELP-01` — Sử dụng `HelpButton` cho tất cả inline help/hint content, không phải custom tooltips
- `RULE-HELP-02` — Sử dụng `renderHTML={true}` khi `helpText` là Quill rich-text HTML string
- `RULE-HELP-03` — Pass plain `ReactNode` cho `helpText` khi là simple text content
- `RULE-HELP-04` — Sử dụng `popperProps` để override placement/offset, không re-wrap với Popper
- `RULE-HELP-05` — Sử dụng `classNameContainer` để style popover box, không phải `className`

#### HorizontalSteps (`uikit-horizontal-steps`)

- `RULE-HSTEPS-01` — `currentStep` là 0-based — không pass giá trị 1-based
- `RULE-HSTEPS-02` — Luôn set `isCanClick` khi `onClickStep` enables navigation
- `RULE-HSTEPS-03` — Không pass `onClickStep` khi steps không navigable
- `RULE-HSTEPS-04` — Giữ length của `stepLabels` ổn định — không conditionally add/remove steps
- `RULE-HSTEPS-05` — Sử dụng `classNameIcon` để style step circles, không phải `className`

#### IconButton (`uikit-icon-button`)

- `RULE-ICON-01` — Chỉ sử dụng trực tiếp khi không cần tooltip
- `RULE-ICON-02` — Pass icon as children, không phải qua `startIcon`/`endIcon`
- `RULE-ICON-03` — Không override `p-0` hoặc `min-w-[unset]`

#### InsertFieldObjectButtonV2 (`uikit-insert-field-object-button-v2`)

- `RULE-INSERT-FIELD-V2-01` — Phải truyền `rootObjects` và `onInsert` — đây là các props bắt buộc
- `RULE-INSERT-FIELD-V2-02` — Cấu hình `rootObjects` đúng định dạng `InsertFieldRootObjectItem`
- `RULE-INSERT-FIELD-V2-03` — Sử dụng `isAdditionalOption` cho các option đặc biệt không phải object
- `RULE-INSERT-FIELD-V2-04` — Sử dụng `children` render prop để tuỳ chỉnh nút bấm
- `RULE-INSERT-FIELD-V2-05` — Sử dụng `isDisabled` để vô hiệu hoá nút
- `RULE-INSERT-FIELD-V2-06` — Sử dụng `selectedDataField` để tự động lọc trường tương thích
- `RULE-INSERT-FIELD-V2-07` — Sử dụng `checkSelectableField` khi cần logic lọc phức tạp
- `RULE-INSERT-FIELD-V2-08` — Sử dụng `filterDataField` để lọc danh sách trường hiển thị — không nhầm với `checkSelectableField`
- `RULE-INSERT-FIELD-V2-09` — Sử dụng `maxLevel` để giới hạn độ sâu duyệt object liên kết
- `RULE-INSERT-FIELD-V2-10` — Sử dụng `description` để thay đổi mô tả trong modal
- `RULE-INSERT-FIELD-V2-11` — Sử dụng `renderName` trong rootObjects để tuỳ chỉnh tên hiển thị
- `RULE-INSERT-FIELD-V2-12` — Kết hợp với `InsertFieldObjectButtonWrapper2` khi cần hiển thị giá trị đã chọn và nút xoá

#### InsertFieldObjectButtonWrapper2 (`uikit-insert-field-object-button-wrapper2`)

- `RULE-INSERT-WRAPPER2-01` — Phải truyền `children` làm nội dung mặc định khi chưa chọn biến
- `RULE-INSERT-WRAPPER2-02` — `selectedVariableSlug` và `onClear` phải đi cùng nhau
- `RULE-INSERT-WRAPPER2-03` — `rootObjects` phải đúng định dạng `InsertFieldRootObjectItem[]`
- `RULE-INSERT-WRAPPER2-04` — Sử dụng `onInsert` để nhận giá trị biến đã chọn — không tự build modal
- `RULE-INSERT-WRAPPER2-05` — Sử dụng `isHideButton` để ẩn nút chèn biến — không ẩn bằng CSS
- `RULE-INSERT-WRAPPER2-06` — Sử dụng `isDisabled` để vô hiệu hoá nút chèn biến và nút clear
- `RULE-INSERT-WRAPPER2-07` — Sử dụng `selectedDataField` để lọc field mặc định
- `RULE-INSERT-WRAPPER2-08` — Sử dụng `checkSelectableField` khi cần logic lọc tuỳ chỉnh
- `RULE-INSERT-WRAPPER2-09` — Sử dụng `maxLevel` để giới hạn độ sâu tree
- `RULE-INSERT-WRAPPER2-10` — Wrapper chiếm toàn bộ chiều rộng parent — bọc trong container để kiểm soát kích thước

#### InsertFieldObjectV2 (`uikit-insert-field-object-v2`)

- `RULE-INSERT-OBJECT-V2-01` — Sử dụng `InsertFieldObjectButtonWrapper2` thay vì tự build UI mở modal
- `RULE-INSERT-OBJECT-V2-02` — Cấu trúc `rootObjects` phải có `slug` và `variableSlug` — không nhầm lẫn
- `RULE-INSERT-OBJECT-V2-03` — Sử dụng `isAdditionalOption` cho các option không phải object
- `RULE-INSERT-OBJECT-V2-04` — Sử dụng `renderName` trong `rootObjects` để tuỳ chỉnh tên hiển thị
- `RULE-INSERT-OBJECT-V2-05` — Sử dụng `selectedDataField` để tự động lọc field tương thích
- `RULE-INSERT-OBJECT-V2-06` — Sử dụng `checkSelectableField` khi cần logic lọc tuỳ chỉnh — trả về đúng kiểu
- `RULE-INSERT-OBJECT-V2-07` — Sử dụng `filterDataField` để lọc field hiển thị — không nhầm với `checkSelectableField`
- `RULE-INSERT-OBJECT-V2-08` — Sử dụng `maxLevel` để giới hạn độ sâu cascader
- `RULE-INSERT-OBJECT-V2-09` — Callback `onInsert` nhận `(value, treeData)` — sử dụng đúng tham số
- `RULE-INSERT-OBJECT-V2-10` — Sử dụng `InsertFieldObjectButtonV2` khi cần tuỳ chỉnh nút mở modal
- `RULE-INSERT-OBJECT-V2-11` — Sử dụng `isHideButton` để ẩn nút mở modal trong Wrapper
- `RULE-INSERT-OBJECT-V2-12` — Sử dụng `isDisabled` để vô hiệu hoá nút mở modal

#### LayoutSelectButton (`uikit-layout-select-button`)

- `RULE-LAYOUT-SELECT-BTN-01` — LayoutSelectButton là controlled component — phải truyền đủ 3 props bắt buộc
- `RULE-LAYOUT-SELECT-BTN-02` — Sử dụng `useGetLayout` hook để lấy dữ liệu — không tự build state và fetch layout
- `RULE-LAYOUT-SELECT-BTN-03` — Không tự build dropdown chọn layout — sử dụng LayoutSelectButton
- `RULE-LAYOUT-SELECT-BTN-04` — Đặt LayoutSelectButton trong container flex justify-end
- `RULE-LAYOUT-SELECT-BTN-05` — Ẩn khi danh sách layout rỗng hoặc đang loading
- `RULE-LAYOUT-SELECT-BTN-06` — Mỗi item trong `listLayouts` phải có `id` và `name` duy nhất

#### LimitableList (`uikit-limitable-list`)

- `RULE-LIMLIST-01` — Cung cấp `getItemWidth` khi `renderItem` trả về nội dung có chiều rộng khác text thuần
- `RULE-LIMLIST-02` — Sử dụng `renderItem` cho dòng chính, `renderListItem` cho popup
- `RULE-LIMLIST-03` — Truyền `showSeparate={false}` khi không muốn dấu chấm phân cách
- `RULE-LIMLIST-04` — Sử dụng `renderListItems` khi cần tuỳ chỉnh toàn bộ nội dung popup
- `RULE-LIMLIST-05` — Sử dụng `popperProps` để tuỳ chỉnh popup, không wrap bằng Popper
- `RULE-LIMLIST-06` — Cung cấp `getTooltipContent` khi item không phải kiểu string

#### ListViewTable (`uikit-list-view-table`)

- `RULE-LIST-VIEW-TABLE-01` — Luôn truyền `dataKey` và `columns` vì đây là props bắt buộc kế thừa từ Table
- `RULE-LIST-VIEW-TABLE-02` — Sử dụng `className` cho container ngoài và `tableClassName` cho Table bên trong — không nhầm lẫn
- `RULE-LIST-VIEW-TABLE-03` — Sử dụng `renderActionButtons` và `renderContextButtons` để hiển thị nút hành động
- `RULE-LIST-VIEW-TABLE-04` — Sử dụng `renderHeadingOptions` để render phần tuỳ chọn giữa header và bảng
- `RULE-LIST-VIEW-TABLE-05` — Sử dụng `title` và `subTitle` cho tiêu đề — không tự tạo heading bên ngoài
- `RULE-LIST-VIEW-TABLE-06` — Kiểm soát `fullHeight` đúng cách để bảng chiếm toàn bộ chiều cao container
- `RULE-LIST-VIEW-TABLE-07` — Kết hợp ListViewTable với TableSettings đúng cách qua render props pattern
- `RULE-LIST-VIEW-TABLE-08` — Không truyền Table props vào sai vị trí — tất cả Table props truyền trực tiếp vào ListViewTable

#### Modal (`uikit-modal`)

- `RULE-MODAL-01` — Conditionally render `<Modal>` để open/close — không có prop `open`/`visible`
- `RULE-MODAL-02` — Sử dụng `renderFooter` với `ModalFooter` cho footer content, không phải manual children
- `RULE-MODAL-03` — Pass `hasDraggable={true}` chỉ khi sử dụng `react-beautiful-dnd` bên trong Modal
- `RULE-MODAL-04` — Sử dụng `size` cho standard widths; sử dụng `width` string chỉ cho custom widths
- `RULE-MODAL-05` — Sử dụng `showHeader={false}` khi không cần header, không phải empty `title`
- `RULE-MODAL-06` — Không explicitly pass default values cho `triggerEscPress` và `hideOnClickOutside`
- `RULE-MODAL-07` — Sử dụng `fixedHeight` chỉ khi modal body phải luôn fill viewport height
- `RULE-MODAL-08` — Sử dụng `placement` cho corner-anchored modals, không phải manual `className` positioning

#### Popper (`uikit-popper`)

- `RULE-POPPER-01` — Sử dụng render prop pattern với `children` (trigger) và `render` (content)
- `RULE-POPPER-02` — Spread `{...params} {...attributes}` trong `render`, không bao giờ đảo ngược thứ tự
- `RULE-POPPER-03` — Sử dụng `PopperMenu` + `PopperMenuItem` cho standard dropdown menus
- `RULE-POPPER-04` — Sử dụng `ref` typed là `PopperRef` để gọi `show`/`hide` từ bên ngoài
- `RULE-POPPER-05` — Sử dụng prop `isOpen` cho controlled mode; không kết hợp với internal `onClick` toggle
- `RULE-POPPER-06` — Sử dụng `allowHideFunction`/`allowShowFunction` để conditionally block show/hide
- `RULE-POPPER-07` — Sử dụng utility `closeAllPopper()` để đóng tất cả poppers cùng lúc
- `RULE-POPPER-08` — Thêm class `hide-on-click` để auto-close popper khi click item
- `RULE-POPPER-09` — Sử dụng `appendTo` khi popper bị clipped bởi `overflow: hidden`
- `RULE-POPPER-10` — Sử dụng `isFixed` khi anchor element có `position: fixed`

#### ResizeBar (`uikit-resize-bar`)

- `RULE-RESIZE-01` — Luôn pass current `width` state vào prop `width`
- `RULE-RESIZE-02` — Update width state qua `onResize`, không phải `onResizeEnd`
- `RULE-RESIZE-03` — `onResizeEnd` nhận `diffWidth` (delta), không phải final width
- `RULE-RESIZE-04` — Chọn `placement` dựa trên panel ở phía nào
- `RULE-RESIZE-05` — Luôn set `minWidth` và `maxWidth` để tránh extreme sizes
- `RULE-RESIZE-06` — Không manually re-implement resize logic với `onResizeStart`/`onResizeEnd`

#### Skeleton (`uikit-skeleton`)

- `RULE-SKEL-01` — Chọn `variant` phù hợp với hình dạng nội dung cần hiển thị
- `RULE-SKEL-02` — Không truyền rõ ràng giá trị mặc định cho `variant`
- `RULE-SKEL-03` — Tuỳ chỉnh kích thước bằng `className`, không dùng inline style
- `RULE-SKEL-04` — Không tự tạo animation loading thủ công, sử dụng Skeleton
- `RULE-SKEL-05` — Tổ hợp nhiều Skeleton để tạo layout loading phức tạp

#### SmartTooltip (`uikit-smart-tooltip`)

- `RULE-SMTTP-01` — Sử dụng khi tooltip chỉ nên xuất hiện khi overflow/clamp
- `RULE-SMTTP-02` — Attach `getRef` vào overflowing element, không phải wrapper
- `RULE-SMTTP-03` — Sử dụng `multipleLines` cho `line-clamp` text
- `RULE-SMTTP-04` — `children` là render prop `({ getRef }) => ReactElement`

#### StepProgressBar (`uikit-step-progress-bar`)

- `RULE-SPB-01` — `percentage` là required prop duy nhất — luôn pass giá trị valid từ 0 đến 100
- `RULE-SPB-02` — Hiểu step circle color behavior: auto-lightened khi `stepBackgroundColor` absent; `stepTextColor` chỉ có hiệu lực khi `stepBackgroundColor` được provide
- `RULE-SPB-03` — Step circle chỉ render khi CẢ `showStepNumber=true` VÀ `stepNumber` được provided
- `RULE-SPB-04` — Bottom info tự hide khi không có content — không set `showBottomInfo={false}` unnecessarily
- `RULE-SPB-05` — `bottomInfo` ưu tiên hơn `bottomInfoText` — không pass cả hai cùng lúc
- `RULE-SPB-06` — Sử dụng prop `color` là single source of truth — không override color qua wrapper styles
- `RULE-SPB-07` — Sử dụng `rightElement` cho avatars hoặc badges bên cạnh title — không build custom wrapper layout

#### Steps (`uikit-steps`)

- `RULE-STEPS-01` — `currentStepIndex` là 0-based — không bao giờ pass giá trị 1-based
- `RULE-STEPS-02` — Tất cả steps đến và bao gồm `currentStepIndex` được highlighted — không có phân biệt "completed vs current"
- `RULE-STEPS-03` — Connector line activates giữa hai adjacent active steps — không chỉ tại current step
- `RULE-STEPS-04` — Không thêm click handlers trên `Steps` itself — navigation phải được managed externally
- `RULE-STEPS-05` — Đảm bảo container đủ rộng — step labels được absolutely positioned và có thể overlap
- `RULE-STEPS-06` — Giữ length của `stepLabels` ổn định — không conditionally add hoặc remove steps

#### StringeeDndContext (`uikit-stringee-dnd-context`)

- `RULE-DND-01` — Luôn sử dụng `useDndHandle` — không manually manage drag state và handlers
- `RULE-DND-02` — `dndId` phải unique across TẤT CẢ containers, không chỉ trong một container
- `RULE-DND-03` — Luôn spread cả `listeners` VÀ `attributes` trên drag handle element
- `RULE-DND-04` — Không spread `listeners` hoặc `attributes` trong `renderOverlay`
- `RULE-DND-05` — `DndContainer` `id` phải khớp chính xác với key trong `items` map
- `RULE-DND-06` — Hiểu `rootContainerIds` — items dragged từ root containers được cloned, không moved
- `RULE-DND-07` — Sử dụng `droppableContainerIds` trên items để restrict containers chúng có thể drop vào
- `RULE-DND-08` — Sử dụng callback `onDragEnd` để persist changes — rely on `activeContainerIdStart` để detect cross-container moves
- `RULE-DND-09` — Pass `hasDraggable={true}` cho `Modal` khi `StringeeDndContext` được render inside nó

#### Table (`uikit-table`)

- `RULE-TABLE-01` — Luôn truyền `dataKey` là đường dẫn đến trường unique identifier
- `RULE-TABLE-02` — Sử dụng `column.render` để tùy chỉnh nội dung cell
- `RULE-TABLE-03` — Truyền `width` cho cột bằng số (đơn vị pixel), không dùng chuỗi
- `RULE-TABLE-04` — Sử dụng `selectable` kết hợp `selectedItems`, `onChangeSelect` và `onChangeSelectAll`
- `RULE-TABLE-05` — Sử dụng `disabledSelect` và `disabledSelectTooltip` để vô hiệu hóa chọn hàng
- `RULE-TABLE-06` — Sử dụng `pinnedColumns` để ghim cột trái/phải
- `RULE-TABLE-07` — Không sử dụng `canReorderRow` cùng với `pinnedColumns` hoặc grouped data
- `RULE-TABLE-08` — Sử dụng `onSortChange` với `sortBy` và `sortDir` cho sắp xếp
- `RULE-TABLE-09` — Sử dụng prop `pagination` tích hợp, không tự render Pagination bên ngoài
- `RULE-TABLE-10` — Truyền `dataSource` dạng object khi sử dụng grouped table
- `RULE-TABLE-11` — Sử dụng `virtualized` cho bảng có nhiều hàng
- `RULE-TABLE-12` — Sử dụng `showingColumns` để ẩn/hiện cột, không lọc mảng `columns`
- `RULE-TABLE-13` — Sử dụng `useTableV2` hook để quản lý state cho Table
- `RULE-TABLE-14` — Sử dụng `border` để cấu hình đường viền, không override bằng className
- `RULE-TABLE-15` — Sử dụng `paramsForRenderCell` để truyền dữ liệu bổ sung vào render
- `RULE-TABLE-16` — Sử dụng `columnResizable` kết hợp `onResizeColumn` cho thay đổi chiều rộng cột

#### TableSettings (`uikit-table-settings`)

- `RULE-TABLE-SETTINGS-01` — Luôn truyền `children` dưới dạng render function nhận các callback từ TableSettings
- `RULE-TABLE-SETTINGS-02` — Sử dụng `useTableSettings` hook để quản lý state và persist vào localStorage
- `RULE-TABLE-SETTINGS-03` — Truyền `storageKey` là chuỗi duy nhất cho mỗi bảng khi dùng `useTableSettings`
- `RULE-TABLE-SETTINGS-04` — Truyền `settings` và `onChangeSettings` đầy đủ và đồng bộ
- `RULE-TABLE-SETTINGS-05` — Dùng `disablePinnedColumn` khi không cần tính năng ghim cột
- `RULE-TABLE-SETTINGS-06` — Sử dụng `text` prop để tuỳ chỉnh nhãn hiển thị trong modal
- `RULE-TABLE-SETTINGS-07` — Sử dụng `onConfirm` để xử lý logic sau khi nhấn Lưu
- `RULE-TABLE-SETTINGS-08` — Sử dụng `ref` và phương thức `reset` để reset cài đặt
- `RULE-TABLE-SETTINGS-09` — Bật `showEditableSelect` khi cần cho phép cấu hình cột editable
- `RULE-TABLE-SETTINGS-10` — Sử dụng `column.canHide` và `column.canChangePosition` để kiểm soát hành vi từng cột
- `RULE-TABLE-SETTINGS-11` — Sử dụng `text.defaultActionFooter` để thêm nội dung tuỳ chỉnh vào footer
- `RULE-TABLE-SETTINGS-12` — Truyền `onResizeColumn` và `onResizeWidth` từ children params xuống Table
- `RULE-TABLE-SETTINGS-13` — Sử dụng `renderSubColumnsSelectButton` cho nút chọn sub-columns
- `RULE-TABLE-SETTINGS-14` — Dùng `column.readOnly` để disable checkbox "Cho phép sửa" cho cột cụ thể

#### Tabs (`uikit-tabs`)

- `RULE-TABS-01` — `Tabs` không có state nội bộ — luôn truyền `currentTabIndex`, `onChange` chỉ cần khi xử lý click chuyển tab
- `RULE-TABS-02` — `currentTabIndex` là 0-based — không bao giờ pass giá trị 1-based
- `RULE-TABS-03` — Sử dụng `scrollable` khi tabs có thể overflow container width
- `RULE-TABS-04` — Sử dụng `underline` để show full-width baseline dưới tất cả tabs
- `RULE-TABS-05` — Sử dụng `layout={TAB_LAYOUT.VERTICAL}` cho vertical tab lists — không sử dụng CSS overrides
- `RULE-TABS-06` — Sử dụng `renderRightIcon` cho actions bên phải — không wrap manually

#### Tag (`uikit-tag`)

- `RULE-TAG-01` — Delete icon yêu cầu CẢ `onDelete` VÀ `showDeleteIcon=true` VÀ `iconClose` phải truthy
- `RULE-TAG-02` — Sử dụng `variant` để control visual style — không override colors qua `className`
- `RULE-TAG-03` — Sử dụng `selected` để toggle active/chosen state — không swap variants
- `RULE-TAG-04` — Sử dụng props `iconLeft` / `iconRight` — không đặt icons as children
- `RULE-TAG-05` — Sử dụng `iconClose` chỉ để replace default × icon — dùng `showDeleteIcon={false}` để hide button
- `RULE-TAG-06` — `disabled` làm tag non-interactive — không combine với `onClick`

#### Tooltip (`uikit-tooltip`)

- `RULE-TTP-01` — Children phải là một React element duy nhất
- `RULE-TTP-02` — Auto-hide trên mobile mặc định (`disabledOnTouchedDevice=true`)
- `RULE-TTP-03` — Sử dụng prop `disabled`, không bao giờ conditionally render `<Tooltip>`
- `RULE-TTP-04` — Ưu tiên `SmartTooltip` cho truncated text
- `RULE-TTP-05` — Không override các pre-configured core styles

#### UsageHelpButton (`uikit-usage-help-button`)

- `RULE-USAGE-HELP-01` — Chỉ sử dụng predefined keys từ `HelpTextVariant` — không pass arbitrary strings
- `RULE-USAGE-HELP-02` — Sử dụng `mode="icon"` cho inline help bên cạnh label — dùng `mode="text"` cho descriptive paragraphs
- `RULE-USAGE-HELP-03` — Không conditionally render dựa trên `helpKey` — component tự return `null` automatically
- `RULE-USAGE-HELP-04` — Sử dụng `textLinkProps` để customize link bên trong help content

#### VerticalSteps (`uikit-vertical-steps`)

- `RULE-VSTEPS-01` — `currentStep` là 0-based — không bao giờ pass giá trị 1-based
- `RULE-VSTEPS-02` — Hiểu ba visual states: past (primary), current (primary), upcoming (gray)
- `RULE-VSTEPS-03` — Luôn set `isCanClick` khi provide `onClickStep` — omit cả hai khi navigation bị disabled
- `RULE-VSTEPS-04` — `VerticalSteps` có fixed dimensions — sử dụng `className` để override cho non-standard layouts
- `RULE-VSTEPS-05` — Giữ length của `stepLabels` ổn định — không conditionally add hoặc remove steps

### Form Components

#### DurationInput (`uikit-duration-input`)

- `RULE-DURATION-01` — Sử dụng `onChange` để nhận giá trị đã format, `onInputChange` để theo dõi giá trị đang nhập
- `RULE-DURATION-02` — Truyền `value` và `defaultValue` dưới dạng chuỗi có đơn vị, không truyền số
- `RULE-DURATION-03` — Chỉ truyền `durationRule` khi cần tuỳ chỉnh đơn vị hoặc tỷ lệ quy đổi
- `RULE-DURATION-04` — Các đơn vị trong giá trị phải theo đúng thứ tự từ lớn đến nhỏ
- `RULE-DURATION-05` — Sử dụng các props kế thừa từ TextField đúng cách
- `RULE-DURATION-06` — Giá trị tự động format khi blur — không tự format bên ngoài

#### DurationRangeInput (`uikit-duration-range-input`)

- `RULE-DRI-01` — Giá trị `value` và `defaultValue` phải là tuple `[string, string]`
- `RULE-DRI-02` — Sử dụng `onChange` để nhận giá trị đã format, `onInputChange` để nhận sự kiện input thô
- `RULE-DRI-03` — Tuỳ chỉnh `durationRule` khi cần thay đổi ký hiệu hoặc quy đổi đơn vị
- `RULE-DRI-04` — Sử dụng controlled hoặc uncontrolled, không trộn lẫn
- `RULE-DRI-05` — Sử dụng `error` và `helperText` để hiển thị lỗi validation
- `RULE-DRI-06` — Sử dụng `readOnly` thay vì `disabled` khi chỉ cần hiển thị giá trị

#### EmailInput (`uikit-email-input`)

- `RULE-EMAIL-01` — Sử dụng `EmailInput` thay vì tự tạo TextField + nút mailto
- `RULE-EMAIL-02` — Luôn truyền `value` để nút mailto hoạt động đúng
- `RULE-EMAIL-03` — Sử dụng `fullWidth` khi cần EmailInput chiếm toàn bộ chiều rộng
- `RULE-EMAIL-04` — Không truyền `endAdornment` vì nút email đã chiếm vị trí bên phải
- `RULE-EMAIL-05` — Sử dụng các prop của TextField cho validation và trạng thái

#### EmojiControlPicker (`uikit-emoji-control-picker`)

- `RULE-EMOJI-PICKER-01` — Luôn truyền `onChange` callback — đây là prop bắt buộc duy nhất
- `RULE-EMOJI-PICKER-02` — Sử dụng `classNames` để tuỳ chỉnh style container
- `RULE-EMOJI-PICKER-03` — Component tự quản lý danh sách frequently used
- `RULE-EMOJI-PICKER-04` — Component đã tích hợp sẵn tìm kiếm
- `RULE-EMOJI-PICKER-05` — Kết hợp với Popper để hiển thị dạng dropdown

#### EmojiPicker (`uikit-emoji-picker`)

- `RULE-EMOJI-01` — Luôn truyền đủ 3 props bắt buộc: `icons`, `value`, `onChange`
- `RULE-EMOJI-02` — Sử dụng `singleSelect` khi chỉ cho phép chọn một emoji
- `RULE-EMOJI-03` — Sử dụng `fullWidth` thay vì wrapper hoặc style thủ công
- `RULE-EMOJI-04` — Sử dụng `disabled` thay vì tự chặn tương tác
- `RULE-EMOJI-05` — Cấu trúc đúng cho `icons` với các loại icon khác nhau
- `RULE-EMOJI-06` — Xoá tất cả giá trị bằng `onChange([])`, không tự reset state

#### FileInput (`uikit-file-input`)

- `RULE-FILEINPUT-01` — Sử dụng `FileInput` thay vì tự tạo input file với nút chọn file
- `RULE-FILEINPUT-02` — Luôn truyền `isMaxFile` để kiểm soát hiển thị component
- `RULE-FILEINPUT-03` — Truyền `tooltip` và `maxSize` để hiển thị thông tin hướng dẫn
- `RULE-FILEINPUT-04` — Truyền `acceptTypes` dưới dạng mảng tên phần mở rộng (không có dấu chấm)
- `RULE-FILEINPUT-05` — Sử dụng `multipleFile` khi cần cho phép chọn nhiều file
- `RULE-FILEINPUT-06` — Sử dụng `iconRight` để thêm nội dung bên phải nút "Chọn file"
- `RULE-FILEINPUT-07` — Reset giá trị input sau khi xử lý onChange

#### FormItem (`uikit-form-item`)

- `RULE-FI-01` — Sử dụng `FormItem` để tạo layout label + component — không tự tạo layout bằng div
- `RULE-FI-02` — Truyền form control vào prop `component` — không truyền vào children
- `RULE-FI-03` — Sử dụng `labelPlacement` để đổi vị trí label
- `RULE-FI-04` — Sử dụng `labelAlign` để căn chỉnh nội dung label
- `RULE-FI-05` — Sử dụng prop `required` để hiển thị dấu sao đỏ — không tự thêm vào label
- `RULE-FI-06` — Sử dụng prop `helperText` để hiển thị tooltip hướng dẫn
- `RULE-FI-07` — Sử dụng prop `readOnly` để hiển thị icon khóa
- `RULE-FI-08` — Sử dụng `rightIcon` và `rightEndComponent` đúng mục đích
- `RULE-FI-09` — Sử dụng `hint` và `subTitle` để hiển thị thông tin phụ
- `RULE-FI-10` — Sử dụng `showLabel={false}` để ẩn label nhưng vẫn giữ hint/subTitle
- `RULE-FI-11` — Sử dụng `labelContainerClassName` để style container của label
- `RULE-FI-12` — Sử dụng `previewComponent` để hiển thị nội dung xem trước

#### InputGroup (`uikit-input-group`)

- `RULE-INPUTGROUP-01` — Truyền `groupSelectProps` đúng cách — bao gồm `value`, `onChange`, `options`
- `RULE-INPUTGROUP-02` — Sử dụng `groupSelectWidth` để điều chỉnh chiều rộng Select
- `RULE-INPUTGROUP-03` — Không truyền rõ ràng giá trị mặc định cho `groupSelectWidth` và `fullWidth`
- `RULE-INPUTGROUP-04` — Sử dụng `fullWidth` khi cần InputGroup chiếm toàn bộ chiều rộng
- `RULE-INPUTGROUP-05` — Children phải là component input có hỗ trợ class `stringee-input-root`
- `RULE-INPUTGROUP-06` — Không truyền `minWidthPopper` cho `groupSelectProps` trừ khi cần ghi đè

#### InputNumber (`uikit-input-number`)

- `RULE-INPUTNUMBER-01` — Sử dụng `format` phù hợp với yêu cầu hiển thị số
- `RULE-INPUTNUMBER-02` — Sử dụng `onChange` với signature `(value: number | null) => void`
- `RULE-INPUTNUMBER-03` — Sử dụng `min`/`max` kết hợp `allowTypeOutsideRange` để kiểm soát giới hạn
- `RULE-INPUTNUMBER-04` — Sử dụng `showIncreaseDecreaseButton={false}` khi không cần nút tăng/giảm
- `RULE-INPUTNUMBER-05` — Sử dụng `fillMissingZero` để tự động điền số 0 vào phần thập phân
- `RULE-INPUTNUMBER-06` — Sử dụng `roundRule` và `precision` để làm tròn số
- `RULE-INPUTNUMBER-07` — Sử dụng `numberOfIntegerDigit` và `numberOfDecimalDigit` để giới hạn số chữ số
- `RULE-INPUTNUMBER-08` — Không truyền rõ ràng giá trị mặc định

#### MarkdownEditor (`uikit-markdown-editor`)

- `RULE-MKEDITOR-01` — Luôn truyền `value` và `onChange` — MarkdownEditor là controlled component
- `RULE-MKEDITOR-02` — Sử dụng `minHeight`/`maxHeight` hoặc `height` cho chiều cao editor
- `RULE-MKEDITOR-03` — Sử dụng `defaultMode` để chọn chế độ hiển thị
- `RULE-MKEDITOR-04` — Sử dụng `showMenu`/`showMarkdown`/`showPreview`/`showFullScreen` để kiểm soát toolbar
- `RULE-MKEDITOR-05` — Sử dụng `handleImageUpload` để upload ảnh
- `RULE-MKEDITOR-06` — Sử dụng `error` và `helperText` cho validation
- `RULE-MKEDITOR-07` — Sử dụng `plugins` để thêm toolbar items
- `RULE-MKEDITOR-08` — Sử dụng `readOnly` để chặn chỉnh sửa

#### MonacoEditor (`uikit-monaco-editor`)

- `RULE-MONACO-01` — Import `MonacoEditor` và `useMonaco` từ ui-kit — không import từ `@monaco-editor/react`
- `RULE-MONACO-02` — Không gọi lại `loader.config` — ui-kit đã cấu hình sẵn
- `RULE-MONACO-03` — Truyền cấu hình editor qua prop `options` — không thao tác DOM trực tiếp
- `RULE-MONACO-04` — Sử dụng prop `height` để đặt chiều cao — không override bằng CSS
- `RULE-MONACO-05` — Sử dụng hook `useMonaco` để truy cập Monaco instance
- `RULE-MONACO-06` — Wrap `MonacoEditor` trong container có chiều rộng xác định

#### MultipleSelect (`uikit-multiple-select`)

- `RULE-MSEL-01` — Luôn truyền đủ 5 props bắt buộc: `value`, `options`, `onChange`, `getLabel`, `getValue`
- `RULE-MSEL-02` — Sử dụng `singleSelect` thay vì tự xây dựng logic chọn một
- `RULE-MSEL-03` — Sử dụng `changeOnBlur` khi muốn trì hoãn thay đổi giá trị
- `RULE-MSEL-04` — Sử dụng `disabledValues` thay vì lọc options để vô hiệu hoá
- `RULE-MSEL-05` — Sử dụng `clearable` và `alsoClearDisabledValues` đúng cách
- `RULE-MSEL-06` — Sử dụng `searchValue` và `onInputChange` cho remote search
- `RULE-MSEL-07` — Sử dụng `renderOption` để tuỳ chỉnh hiển thị option
- `RULE-MSEL-08` — Sử dụng `renderTag` để tuỳ chỉnh hiển thị tag
- `RULE-MSEL-09` — Sử dụng `hideAfterSelect` khi muốn đóng dropdown sau mỗi lần chọn
- `RULE-MSEL-10` — Sử dụng `maxSelectedValue` để giới hạn số lượng lựa chọn
- `RULE-MSEL-11` — Sử dụng `hasSelectAll` để hiển thị option "Chọn tất cả"
- `RULE-MSEL-12` — Sử dụng `isShowIconSearch` khi muốn hiển thị icon search
- `RULE-MSEL-13` — Sử dụng `onScrollToBottom` cho lazy loading
- `RULE-MSEL-14` — Không truyền rõ ràng các giá trị mặc định
- `RULE-MSEL-15` — Sử dụng `renderLimitTag` và `renderLimitValue` để tuỳ chỉnh tag bị giới hạn
- `RULE-MSEL-16` — Sử dụng `startAdornment` và `alwaysShowAdornment` để thêm icon/text phía trước

#### PhoneInput (`uikit-phone-input`)

- `RULE-PHONE-01` — Sử dụng `PhoneInput` thay vì tự tạo TextField + CountrySelect
- `RULE-PHONE-02` — Luôn truyền `value` và `onChange` để component hoạt động đúng
- `RULE-PHONE-03` — Sử dụng `isShowButtonCall` hoặc `isShowButtonCallWhenHover` cho nút gọi điện
- `RULE-PHONE-04` — Sử dụng `handleCall` khi cần tuỳ chỉnh hành vi gọi điện
- `RULE-PHONE-05` — Sử dụng `countries` để giới hạn danh sách quốc gia
- `RULE-PHONE-06` — Sử dụng `fullWidth` khi cần PhoneInput chiếm toàn bộ chiều rộng
- `RULE-PHONE-07` — Không truyền `startAdornment` vì CountrySelect đã chiếm vị trí bên trái
- `RULE-PHONE-08` — Sử dụng `autoFillDialCode` để tự động điền mã quốc gia
- `RULE-PHONE-09` — Sử dụng các prop của TextField cho validation và trạng thái
- `RULE-PHONE-10` — Sử dụng helper `validatePhoneNumber` để kiểm tra số điện thoại

#### Radio (`uikit-radio`)

- `RULE-RADIO-01` — Pair `checked` với `onChange` trong controlled mode — omit `checked` cho uncontrolled
- `RULE-RADIO-02` — Sử dụng `size` cho kích thước — không override width/height qua className
- `RULE-RADIO-03` — Không truyền `type` — component luôn là radio
- `RULE-RADIO-04` — Sử dụng `RadioGroup` để quản lý nhóm radio
- `RULE-RADIO-05` — Sử dụng `FormControlLabel` để thêm label cho radio
- `RULE-RADIO-06` — Sử dụng `row` trên RadioGroup cho layout ngang
- `RULE-RADIO-07` — `disabled` trên RadioGroup override disabled của từng children
- `RULE-RADIO-08` — Mỗi Radio trong RadioGroup phải có `value` unique
- `RULE-RADIO-09` — Sử dụng `labelPlacement` trên FormControlLabel để đổi vị trí label
- `RULE-RADIO-10` — `value` hỗ trợ `number | string | boolean` — không cần ép kiểu thủ công

#### RadioGroup (`uikit-radio-group`)

- `RULE-RADIO-01` — RadioGroup luôn là controlled — phải truyền `value` và `onChange`
- `RULE-RADIO-02` — Sử dụng `RadioGroup` để quản lý nhóm radio — không tự viết logic chọn
- `RULE-RADIO-03` — Sử dụng `FormControlLabel` để thêm label cho radio
- `RULE-RADIO-04` — Sử dụng `row` trên RadioGroup cho layout ngang
- `RULE-RADIO-05` — `disabled` trên RadioGroup override disabled của từng children
- `RULE-RADIO-06` — Mỗi Radio trong RadioGroup phải có `value` unique
- `RULE-RADIO-07` — Sử dụng `size` trên Radio cho kích thước
- `RULE-RADIO-08` — Không truyền `type` — component luôn là radio
- `RULE-RADIO-09` — Sử dụng `labelPlacement` trên FormControlLabel để đổi vị trí label
- `RULE-RADIO-10` — `onChange` trả về giá trị của radio được chọn — không lấy từ event

#### Rating (`uikit-rating`)

- `RULE-RATING-01` — Pair `value` với `onChange` trong controlled mode
- `RULE-RATING-02` — Omit `value` và `onChange` cho uncontrolled; dùng `defaultValue`
- `RULE-RATING-03` — `onChange` trả về `number | null`, không phải event
- `RULE-RATING-04` — Sử dụng prop `icon` để tuỳ chỉnh icon
- `RULE-RATING-05` — Sử dụng `max` để thay đổi số lượng icon
- `RULE-RATING-06` — Sử dụng `iconSize` và `gap` để tuỳ chỉnh kích thước
- `RULE-RATING-07` — Sử dụng `direction` để thay đổi hướng hiển thị
- `RULE-RATING-08` — Sử dụng `disabled` để vô hiệu hoá tương tác

#### Reaction (`uikit-reaction`)

- `RULE-REACTION-01` — Sử dụng options mặc định khi chỉ cần 5 emoji cơ bản
- `RULE-REACTION-02` — Truyền `options` đúng format `ReactionOption` khi cần tuỳ chỉnh
- `RULE-REACTION-03` — Sử dụng controlled (`value` + `onChange`) hoặc uncontrolled (`defaultValue`)
- `RULE-REACTION-04` — Sử dụng prop `direction` để thay đổi hướng hiển thị
- `RULE-REACTION-05` — Sử dụng `gap` và `iconSize` để tuỳ chỉnh khoảng cách
- `RULE-REACTION-06` — Xử lý giá trị `null` trong `onChange` khi người dùng bỏ chọn

#### SearchInput (`uikit-search-input`)

- `RULE-SINPUT-01` — Luôn truyền `value` và `onChange` — đây là controlled component
- `RULE-SINPUT-02` — Sử dụng prop `onClear` để hiển thị nút xoá
- `RULE-SINPUT-03` — Chỉ truyền `iconClear` khi cần thay đổi icon xoá mặc định
- `RULE-SINPUT-04` — Sử dụng `className` để tuỳ chỉnh kích thước và style
- `RULE-SINPUT-05` — Nút xoá chỉ hiển thị khi có cả `onClear` và `value`

#### Select (`uikit-select`)

- `RULE-SELECT-01` — Select là controlled component — phải truyền `value` và `onChange`
- `RULE-SELECT-02` — Sử dụng `getLabel` và `getValue` khi option không có dạng `{ label, value }`
- `RULE-SELECT-03` — Sử dụng `mode="group"` kết hợp `groupOptions` cho option theo nhóm
- `RULE-SELECT-04` — Sử dụng `searchable` để bật tìm kiếm
- `RULE-SELECT-05` — Sử dụng `filterBySearch={false}` khi muốn tự lọc option bên ngoài
- `RULE-SELECT-06` — Sử dụng `clearable` để hiển thị nút xoá
- `RULE-SELECT-07` — Sử dụng `error` kết hợp `helperText` cho validation
- `RULE-SELECT-08` — Sử dụng `renderOption` và `renderValue` để tuỳ chỉnh hiển thị
- `RULE-SELECT-09` — `disabled` và `readOnly` có hành vi khác nhau
- `RULE-SELECT-10` — Sử dụng `disabledValues` để vô hiệu hoá option cụ thể
- `RULE-SELECT-11` — Sử dụng `fullWidth` thay vì override className cho chiều rộng
- `RULE-SELECT-12` — Sử dụng `onScrollToBottom` cho lazy loading
- `RULE-SELECT-13` — Sử dụng `hidePopperOnSelect={false}` khi không muốn đóng dropdown
- `RULE-SELECT-14` — Sử dụng `isEqual` khi cần so sánh option tuỳ chỉnh
- `RULE-SELECT-15` — Sử dụng `height` để thay đổi chiều cao input

#### Slider (`uikit-slider`)

- `RULE-SLIDER-01` — Pair `value` với `onChange` trong controlled mode
- `RULE-SLIDER-02` — Omit `value` cho uncontrolled usage; KHÔNG pass `defaultValue`
- `RULE-SLIDER-03` — `className` style outer wrapper, không phải input
- `RULE-SLIDER-04` — Sử dụng `min`, `max`, và `step` cho range control; không tính toán lại externally
- `RULE-SLIDER-05` — Không bao giờ pass `type` — nó luôn là `"range"`

#### Switch (`uikit-switch`)

- `RULE-SWITCH-01` — Pair `checked` với `onChange` trong controlled mode — omit `checked` cho uncontrolled
- `RULE-SWITCH-02` — Sử dụng `size` cho kích thước — không override width/height qua className
- `RULE-SWITCH-03` — Không truyền `type` — component luôn là checkbox
- `RULE-SWITCH-04` — Không tự build giao diện switch bằng div/span — sử dụng component Switch
- `RULE-SWITCH-05` — Sử dụng `disabled` prop — không override style disabled qua className
- `RULE-SWITCH-06` — Không sử dụng `Checkbox` khi ngữ nghĩa là bật/tắt — dùng `Switch`

#### TagsInput (`uikit-tags-input`)

- `RULE-TAGSINPUT-01` — Khi sử dụng với object, phải truyền `getLabel` và `getValue`
- `RULE-TAGSINPUT-02` — Sử dụng `submitWithStringValue={false}` khi chỉ cho phép chọn từ suggest list
- `RULE-TAGSINPUT-03` — Bật `searchable` và `filterBySearch` đồng thời để lọc suggest list
- `RULE-TAGSINPUT-04` — Sử dụng `renderSuggestItem` khi dùng `suggestListOptions` với kiểu object
- `RULE-TAGSINPUT-05` — Sử dụng `clearable` kết hợp `onClearable` để xử lý logic khi xoá tất cả
- `RULE-TAGSINPUT-06` — Sử dụng `isChangeOverlap` để cho phép tag trùng lặp
- `RULE-TAGSINPUT-07` — Sử dụng `renderTag` để tuỳ chỉnh giao diện tag
- `RULE-TAGSINPUT-08` — Truyền `startAdornment`/`endAdornment` đúng cách
- `RULE-TAGSINPUT-09` — `error` tự động set `color="error"` — không cần truyền cả hai
- `RULE-TAGSINPUT-10` — Sử dụng `appendToBody={false}` khi cần suggest list nằm trong container cha

#### TextField (`uikit-text-field`)

- `RULE-TF-01` — Sử dụng `multipleLine` để chuyển sang textarea, không tự render `<textarea>`
- `RULE-TF-02` — Truyền icons/elements qua `startAdornment`/`endAdornment`
- `RULE-TF-03` — Sử dụng `readOnly` để hiển thị chế độ chỉ đọc
- `RULE-TF-04` — Sử dụng `htmlReadOnly` khi cần input không cho nhập nhưng vẫn giữ giao diện
- `RULE-TF-05` — Sử dụng `error` kết hợp `helperText` để hiển thị lỗi
- `RULE-TF-06` — Sử dụng `inputClassName` để tuỳ chỉnh style input
- `RULE-TF-07` — Sử dụng `hideValue` thay vì `type="password"` để ẩn giá trị
- `RULE-TF-08` — Sử dụng `defaultExpand` để textarea tự động giãn theo nội dung
- `RULE-TF-09` — Không truyền rõ ràng giá trị mặc định cho `showBorderOnBlur`, `rows`, `resize`
- `RULE-TF-10` — Sử dụng `hideMaxLength` để ẩn bộ đếm ký tự

#### TextLink (`uikit-text-link`)

- `RULE-TEXTLINK-01` — Sử dụng prop `component` để tích hợp với router, không wrap bằng Link
- `RULE-TEXTLINK-02` — Truyền icons qua `startIcon`/`endIcon`, không đặt trong children
- `RULE-TEXTLINK-03` — Sử dụng `size` phù hợp với ngữ cảnh
- `RULE-TEXTLINK-04` — Không truyền rõ ràng giá trị mặc định cho `size`, `underline` và `component`
- `RULE-TEXTLINK-05` — Sử dụng `underline` khi cần nhấn mạnh tính tương tác
- `RULE-TEXTLINK-06` — Luôn truyền `target="_blank"` khi link mở trang ngoài

#### TimePicker (`uikit-time-picker`)

- `RULE-TIME-PICKER-01` — `value` là `Dayjs | null` — không truyền Date, string, hay timestamp
- `RULE-TIME-PICKER-02` — Sử dụng `format` để định dạng hiển thị — mặc định `"HH:mm:ss"`
- `RULE-TIME-PICKER-03` — Sử dụng `min`/`max` là `Dayjs` để giới hạn khoảng thời gian
- `RULE-TIME-PICKER-04` — Sử dụng `renderValue` để tuỳ chỉnh hiển thị
- `RULE-TIME-PICKER-05` — Sử dụng `timeSelectProps` để tuỳ chỉnh TimeSelect
- `RULE-TIME-PICKER-06` — Input hỗ trợ nhập tay — blur sẽ parse và validate theo `format`
- `RULE-TIME-PICKER-07` — Sử dụng `hidePopperOnClear` để đóng popup khi xoá giá trị
- `RULE-TIME-PICKER-08` — Sử dụng `helperText` kết hợp `error` để hiển thị thông báo lỗi
- `RULE-TIME-PICKER-09` — Sử dụng `popperProps` để tuỳ chỉnh Popper
- `RULE-TIME-PICKER-10` — Không tự build icon — component đã tích hợp sẵn

#### TimeRangePicker (`uikit-time-range-picker`)

- `RULE-TIME-RANGE-PICKER-01` — `value` là tuple `[Dayjs | null, Dayjs | null]`
- `RULE-TIME-RANGE-PICKER-02` — `onChange` nhận tuple `[Dayjs | null, Dayjs | null]`
- `RULE-TIME-RANGE-PICKER-03` — Sử dụng `fromMin`/`fromMax`/`toMin`/`toMax` để giới hạn thời gian
- `RULE-TIME-RANGE-PICKER-04` — Sử dụng `format` để thay đổi format hiển thị
- `RULE-TIME-RANGE-PICKER-05` — Sử dụng `renderValue` để tuỳ chỉnh hiển thị
- `RULE-TIME-RANGE-PICKER-06` — Sử dụng `timeSelectProps` để tuỳ chỉnh TimeSelect
- `RULE-TIME-RANGE-PICKER-07` — Sử dụng `error` + `helperText` cho validation
- `RULE-TIME-RANGE-PICKER-08` — Sử dụng `getFromRef` và `getToRef` để truy cập input element
- `RULE-TIME-RANGE-PICKER-09` — Clear giá trị bằng `[null, null]` — icon clear đã tích hợp
- `RULE-TIME-RANGE-PICKER-10` — Input hỗ trợ nhập tay — blur sẽ parse và validate

#### TimeSelect (`uikit-time-select`)

- `RULE-TIME-SELECT-01` — `value` là `Dayjs | null` — không truyền Date, string, hay timestamp
- `RULE-TIME-SELECT-02` — Sử dụng `format` để thay đổi hiển thị — không tự format bên ngoài
- `RULE-TIME-SELECT-03` — Sử dụng `min`/`max` là `Dayjs` để giới hạn khoảng thời gian
- `RULE-TIME-SELECT-04` — Sử dụng `height` để thay đổi chiều cao danh sách
- `RULE-TIME-SELECT-05` — Sử dụng `fullWidth` để mở rộng chiều ngang
- `RULE-TIME-SELECT-06` — Không tự build danh sách chọn giờ — sử dụng TimeSelect
- `RULE-TIME-SELECT-07` — Ưu tiên dùng DatePicker với format có token giờ thay vì kết hợp riêng TimeSelect

#### TreeSelect (`uikit-tree-select`)

- `RULE-TREE-SELECT-01` — Luôn truyền đủ 4 props bắt buộc: `value`, `onChange`, `modeSelectTreeData`, `options`
- `RULE-TREE-SELECT-02` — Sử dụng `treeProps.fieldNames` để ánh xạ dữ liệu
- `RULE-TREE-SELECT-03` — Chọn đúng `modeSelectTreeData` theo yêu cầu nghiệp vụ
- `RULE-TREE-SELECT-04` — Sử dụng `clearable` và `onClear` để xoá giá trị
- `RULE-TREE-SELECT-05` — Sử dụng `showSearch` để bật tìm kiếm
- `RULE-TREE-SELECT-06` — Sử dụng `inputProps` cho validation
- `RULE-TREE-SELECT-07` — Sử dụng `maxLength` để giới hạn số lượng chọn
- `RULE-TREE-SELECT-08` — Sử dụng `disableNodeIds` để vô hiệu hoá node cụ thể
- `RULE-TREE-SELECT-09` — Sử dụng `usePoper={false}` khi cần nhúng tree trực tiếp
- `RULE-TREE-SELECT-10` — Truyền `selectedOptions` khi giá trị đã chọn có thể không nằm trong `options`
- `RULE-TREE-SELECT-11` — Sử dụng `showOperatorSelect` cho bộ lọc toán tử
- `RULE-TREE-SELECT-12` — Sử dụng `showBorderOnBlur={false}` để ẩn border khi blur

#### UrlInput (`uikit-url-input`)

- `RULE-URL-01` — Sử dụng `UrlInput` thay vì tự tạo TextField + nút mở link
- `RULE-URL-02` — Luôn truyền `value` để nút mở URL hoạt động đúng
- `RULE-URL-03` — Sử dụng `hasDisplayText` khi cần hiển thị text thay thế cho URL
- `RULE-URL-04` — Truyền `autoFillDisplayText={false}` khi không muốn tự động lấy title từ URL
- `RULE-URL-05` — Sử dụng `fullWidth` khi cần UrlInput chiếm toàn bộ chiều rộng
- `RULE-URL-06` — Sử dụng `wrapperClassName` để style container ngoài
- `RULE-URL-07` — Sử dụng `displayTextInputProps` để tùy chỉnh input display text
- `RULE-URL-08` — Popper display text tự động bị vô hiệu hóa khi `disabled` hoặc `readOnly`

### Hooks

#### useForceUpdate (`uikit-use-force-update`)

- `RULE-FU-01` — Sử dụng `useForceUpdate` thay vì tự tạo state counter
- `RULE-FU-02` — Sử dụng `forceUpdateState` làm dependency khi cần trigger effect sau re-render
- `RULE-FU-03` — Chỉ sử dụng khi React dependency tracking không đủ — ưu tiên quản lý state đúng cách

#### useGetDataFieldBySlug (`uikit-use-get-data-field-by-slug`)

- `RULE-GDFBS-01` — Ưu tiên truyền `objectSlug` — chỉ dùng `objectId` khi không có slug
- `RULE-GDFBS-02` — Ưu tiên `dataFieldSlugs` — hook bỏ qua `dataFieldIds` nếu slugs được truyền
- `RULE-GDFBS-03` — Kết quả giữ nguyên thứ tự mảng đầu vào — null nếu không tìm thấy
- `RULE-GDFBS-04` — Không tự fetch object và tìm field — hook đã tích hợp `useQueryObjectDetailBySlug`
- `RULE-GDFBS-05` — Kiểm tra `isLoading` trước khi sử dụng `fields`

#### useGetDataFieldOptionLabel (`uikit-use-get-data-field-option-label`)

- `RULE-GDFOL-01` — Luôn truyền `objectSlug` — đây là prop bắt buộc duy nhất
- `RULE-GDFOL-02` — Sử dụng `getOptionLabel` thay vì tự tìm kiếm option trong fields
- `RULE-GDFOL-03` — Hàm trả về chuỗi rỗng khi không tìm thấy — không cần kiểm tra null
- `RULE-GDFOL-04` — Callback đã được memoize — an toàn khi dùng trong dependency array

#### useGetFieldLabel (`uikit-use-get-field-label`)

- `RULE-GFL-01` — Luôn truyền đủ `objectSlug` và `fieldSlugs` — đây là 2 props bắt buộc
- `RULE-GFL-02` — Không tự fetch object và tìm field name — hook đã tích hợp API
- `RULE-GFL-03` — Kết quả giữ nguyên thứ tự mảng đầu vào — chuỗi rỗng cho slug không tìm thấy
- `RULE-GFL-04` — Truyền `hasTranslate={false}` khi cần tên gốc (chưa dịch)

#### useGetLayout (`uikit-use-get-layout`)

- `RULE-GL-01` — Luôn truyền đủ `functionLabel` và `objectSlug` — đây là 2 props bắt buộc
- `RULE-GL-02` — Sử dụng `"ADD"` cho form tạo mới, `"VIEW"` cho form xem/sửa
- `RULE-GL-03` — Sử dụng `hiddenFieldSlugs` để ẩn fields — không tự lọc layout
- `RULE-GL-04` — Sử dụng `readOnlyFieldSlugs` để đánh dấu fields chỉ đọc
- `RULE-GL-05` — Kết hợp với `LayoutSelectButton` để chuyển layout
- `RULE-GL-06` — Kiểm tra `isLoading` trước khi render FormBuilder
- `RULE-GL-07` — Sử dụng `layoutId` khi cần layout cụ thể

#### useGetRecordPageLink (`uikit-use-get-record-page-link`)

- `RULE-GRPL-01` — Luôn sử dụng hook thay vì hardcode route — tự phát hiện serial page context
- `RULE-GRPL-02` — Sử dụng đúng hàm: `listPageLink`, `createPageLink`, `viewPageLink`
- `RULE-GRPL-03` — Không tự kiểm tra pathname — hook đã tự phát hiện `/s/` hoặc `/c/s/`
- `RULE-GRPL-04` — Không tự sử dụng `generatePath` và `routeMapFullPath` — hook đã wrap sẵn

#### useGetStateFromSearchParams (`uikit-use-get-state-from-search-params`)

- `RULE-GSSP-01` — Truyền hàm `getState` để chuyển đổi search params thành typed state
- `RULE-GSSP-02` — Không tự parse `location.search` — hook reactive với URL
- `RULE-GSSP-03` — Kết hợp với `useSaveToSearchParams` để tạo two-way binding URL ↔ State
- `RULE-GSSP-04` — Hàm `getState` nên cung cấp giá trị mặc định cho mỗi field

#### useHistoriesState (`uikit-use-histories-state`)

- `RULE-HS-01` — Sử dụng `push` thêm state mới, `replace` cập nhật state hiện tại — không nhầm
- `RULE-HS-02` — Kiểm tra `canBack`/`canForward` trước khi hiển thị nút undo/redo
- `RULE-HS-03` — Bật `enableShortcuts` để hỗ trợ Ctrl/Cmd+Z và Ctrl/Cmd+Shift+Z
- `RULE-HS-04` — Sử dụng `toggleShortcuts` để tạm tắt khi component khác cần phím tắt
- `RULE-HS-05` — Hook tự loại bỏ state trùng lặp — không cần kiểm tra trước khi push
- `RULE-HS-06` — Không tự implement undo/redo — sử dụng hook này

#### useMergeRecordFilters (`uikit-use-merge-record-filters`)

- `RULE-MRF-01` — Sử dụng `mergeRecordFilters` thay vì tự merge — hook xử lý logic expression tự động
- `RULE-MRF-02` — Filters với `logicType: "NONE"` tự động bị loại bỏ
- `RULE-MRF-03` — Truyền `undefined` trong mảng filters được phép — hook tự bỏ qua
- `RULE-MRF-04` — Kết quả luôn có cấu trúc hợp lệ — object rỗng khi không có filter
- `RULE-MRF-05` — Sử dụng `logicType: "OR"` khi cần bất kỳ điều kiện nào thỏa mãn

#### useMergedRefs (`uikit-use-merge-refs`)

- `RULE-MREFS-01` — Sử dụng `useMergedRefs` khi cần gán nhiều refs cho cùng element
- `RULE-MREFS-02` — Hook hỗ trợ cả function refs và object refs
- `RULE-MREFS-03` — Sử dụng phổ biến trong `forwardRef` components
- `RULE-MREFS-04` — Tên export là `useMergedRefs` (có "d") — không phải `useMergeRefs`

#### useReplaceRecordValue (`uikit-use-replace-record-value`)

- `RULE-RRV-01` — Template variables format: `{$variableName.path.to.field}` — `$` bắt buộc
- `RULE-RRV-02` — `variableRecordMap` key phải khớp tên biến trong template (bao gồm `$`)
- `RULE-RRV-03` — Hàm là async — phải await hoặc .then()
- `RULE-RRV-04` — Hook tự batch API calls và cache — không cần tối ưu bên ngoài
- `RULE-RRV-05` — Không tự fetch và replace — hook render giá trị theo đúng kiểu field

#### useSaveToSearchParams (`uikit-use-save-to-search-params`)

- `RULE-STSP-01` — Truyền object data — hook tự merge và loại bỏ giá trị rỗng
- `RULE-STSP-02` — Không tự cập nhật URL — hook dùng debounce 800ms và replace history
- `RULE-STSP-03` — Sử dụng `disabled` để tạm dừng cập nhật URL
- `RULE-STSP-04` — Kết hợp với `useGetStateFromSearchParams` cho two-way binding
- `RULE-STSP-05` — Hook bỏ qua lần render đầu tiên

#### useSyncState (`uikit-use-sync-state`)

- `RULE-SS-01` — API giống `useState` — destructure thành `[state, setState]`
- `RULE-SS-02` — `key` phải duy nhất — channel name là `sync-state-{key}`
- `RULE-SS-03` — Không tự implement BroadcastChannel — hook xử lý cleanup
- `RULE-SS-04` — Khi `value` prop thay đổi, state local tự cập nhật
- `RULE-SS-05` — Chỉ hoạt động giữa các tab cùng origin

## Rule Files

```
rules/tokens-tailwind-config.md
rules/color-palette-usage.md
rules/typography-prose-classes.md
rules/spacing-rem-only.md
rules/cx-class-grouping.md
rules/uikit-accordion.md
rules/uikit-alert.md
rules/uikit-avatar.md
rules/uikit-avatar-field.md
rules/uikit-breadcrumbs.md
rules/uikit-button.md
rules/uikit-button-group.md
rules/uikit-calendar.md
rules/uikit-cascader.md
rules/uikit-checkbox.md
rules/uikit-checkbox-group.md
rules/uikit-circular-progress-bar.md
rules/uikit-ckeditor.md
rules/uikit-collapsible-container.md
rules/uikit-color-picker.md
rules/uikit-confirm-modal.md
rules/uikit-copy-button.md
rules/uikit-copy-link-button.md
rules/uikit-country-select.md
rules/uikit-created-by-info.md
rules/uikit-date-picker.md
rules/uikit-date-range-calendar.md
rules/uikit-date-range-picker.md
rules/uikit-date-time-with-tooltip.md
rules/uikit-draggable-item.md
rules/uikit-draggable-wrapper.md
rules/uikit-drawer.md
rules/uikit-duration-input.md
rules/uikit-duration-range-input.md
rules/uikit-email-input.md
rules/uikit-emoji-control-picker.md
rules/uikit-emoji-picker.md
rules/uikit-file-input.md
rules/uikit-file-preview.md
rules/uikit-filter-generator.md
rules/uikit-filter-generator-base.md
rules/uikit-form-control-label.md
rules/uikit-form-item.md
rules/uikit-help-button.md
rules/uikit-helper-text.md
rules/uikit-horizontal-steps.md
rules/uikit-icon-button.md
rules/uikit-insert-field-object-button-v2.md
rules/uikit-insert-field-object-button-wrapper2.md
rules/uikit-insert-field-object-v2.md
rules/uikit-input-group.md
rules/uikit-input-number.md
rules/uikit-layout-select-button.md
rules/uikit-lazy-image.md
rules/uikit-limitable-list.md
rules/uikit-list-view-table.md
rules/uikit-markdown-editor.md
rules/uikit-markdown-preview.md
rules/uikit-modal.md
rules/uikit-monaco-editor.md
rules/uikit-multiple-select.md
rules/uikit-pagination.md
rules/uikit-personnel-detail-card.md
rules/uikit-personnel-detail-popper.md
rules/uikit-phone-input.md
rules/uikit-popper.md
rules/uikit-popper-record-info.md
rules/uikit-radio.md
rules/uikit-radio-group.md
rules/uikit-rating.md
rules/uikit-reaction.md
rules/uikit-resize-bar.md
rules/uikit-search-input.md
rules/uikit-select.md
rules/uikit-show.md
rules/uikit-skeleton.md
rules/uikit-slider.md
rules/uikit-smart-tooltip.md
rules/uikit-spinner.md
rules/uikit-step-progress-bar.md
rules/uikit-steps.md
rules/uikit-stringee-dnd-context.md
rules/uikit-switch.md
rules/uikit-table.md
rules/uikit-table-settings.md
rules/uikit-tabs.md
rules/uikit-tag.md
rules/uikit-tags-input.md
rules/uikit-text-field.md
rules/uikit-text-link.md
rules/uikit-time-picker.md
rules/uikit-time-range-picker.md
rules/uikit-time-select.md
rules/uikit-toast-message.md
rules/uikit-toggle-collapse-button.md
rules/uikit-tooltip.md
rules/uikit-tree-select.md
rules/uikit-url-input.md
rules/uikit-usage-help-button.md
rules/uikit-use-responsive.md
rules/uikit-use-convert-page-settings.md
rules/uikit-use-force-update.md
rules/uikit-use-get-data-field-by-slug.md
rules/uikit-use-get-data-field-option-label.md
rules/uikit-use-get-field-label.md
rules/uikit-use-get-layout.md
rules/uikit-use-get-record-page-link.md
rules/uikit-use-get-state-from-search-params.md
rules/uikit-use-histories-state.md
rules/uikit-use-merge-record-filters.md
rules/uikit-use-merge-refs.md
rules/uikit-use-replace-record-value.md
rules/uikit-use-save-to-search-params.md
rules/uikit-use-sync-state.md
rules/uikit-vertical-steps.md
rules/uikit-import-source.md
rules/uikit-text-vietnamese-first.md
rules/uikit-parallel-sub-agents.md
rules/uikit-no-auto-commit.md
rules/uikit-form-react-hook-form.md
rules/uikit-object-field-record.md
rules/no-typescript-any.md
rules/prettier-and-lint-check.md
rules/no-auto-markdown.md
rules/page-list-resource.md
rules/dayjs-date-time.md
```

---

# Create Commit

Quy trình tạo git commit tuân theo quy ước commit message của Cogover.

## Khi nào áp dụng

Sử dụng khi:
- Được yêu cầu "tạo commit" hoặc "create commit"
- Cần stage và commit các thay đổi với định dạng message đúng
- Làm việc trên các nhánh có hoặc không có mã task SPT

## Cách sử dụng

Đọc file `AGENTS.md` phần **Hướng dẫn tạo Commit** để xem quy trình commit đầy đủ và các quy tắc.

---

# i18n

Hướng dẫn đầy đủ để dịch text tiếng Việt trong các dự án React sử dụng react-i18next.

## Khi nào áp dụng

Sử dụng khi:
- Được yêu cầu "dịch các text tiếng Việt" (translate Vietnamese texts)
- Cần thay thế hardcoded Vietnamese strings bằng i18n translation calls
- Làm việc với plain strings, strings có placeholders, hoặc HTML templates

## Cách sử dụng

Đọc file `AGENTS.md` phần **Hướng dẫn Dịch i18n cho UI** để xem quy trình dịch đầy đủ và các quy tắc.
