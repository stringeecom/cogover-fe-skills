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
