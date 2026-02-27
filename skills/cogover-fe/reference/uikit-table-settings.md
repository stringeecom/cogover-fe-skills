## Cách sử dụng TableSettings Component của ui-kit

Path: `ui-kit/src/components/TableSettings`
Hook: `ui-kit/src/components/TableSettings/hooks/useTableSettings`

### Props

| Prop | Bắt buộc | Mô tả |
|---|---|---|
| `dataKey` | ✓ | Field unique identifier |
| `settings` | ✓ | `{ columns, showingColumns, pinnedColumns, editableColumns? }` |
| `onChangeSettings` | ✓ | `{ columns, showingColumns, pinnedColumns, editableColumns?, selectedItems? }` |
| `children` | ✓ | Render function `({ showSettings, onChangeSelect, onChangeSelectAll, onResizeColumn, onResizeWidth }) => ReactNode` |
| `text` | — | Tuỳ chỉnh nhãn modal: `{ title, searchPlaceholder, pinnedLeftColumnTitle, pinnedMiddleColumnTitle, pinnedRightColumnTitle, confirmLabel, cancelLabel, defaultActionFooter, helper }` |
| `disablePinnedColumn` | — | Ẩn tính năng ghim cột |
| `showEditableSelect` | — | Hiển thị checkbox "Cho phép sửa" |
| `renderSubColumnsSelectButton` | — | Render nút chọn sub-columns cho cột lookup |
| `onConfirm` | — | Callback khi nhấn Lưu: `(settings) => void` |
| `ref` | — | `TableSettingsRef` — expose method `reset(params?)` |

### Basic usage

```tsx
const {
  columns, setColumns, showingColumns, setShowingColumns,
  pinnedColumns, setPinnedColumns, sortBy, setSortBy, sortDir, setSortDir,
} = useTableSettings<RecordItem>({
  storageKey: "user-list-settings",    // unique per table, key: table-setting-{storageKey}
  initState: {
    initColumns,
    initPinnedColumns: { left: ["name"], right: [] },
    initShowingColumns: ["name", "email", "status"],
  },
});

<TableSettings
  dataKey="id"
  settings={{ columns, showingColumns, pinnedColumns }}
  onChangeSettings={{ columns: setColumns, showingColumns: setShowingColumns, pinnedColumns: setPinnedColumns }}
>
  {({ showSettings, onChangeSelect, onChangeSelectAll, onResizeColumn, onResizeWidth }) => (
    <Table
      dataSource={data}
      dataKey="id"
      columns={columns}
      showingColumns={showingColumns}
      pinnedColumns={pinnedColumns}
      columnResizable
      selectable
      selectedItems={selectedItems}
      onChangeSelect={onChangeSelect}
      onChangeSelectAll={onChangeSelectAll}
      onResizeColumn={onResizeColumn}
      onResizeWidth={onResizeWidth}
    />
  )}
</TableSettings>
```

### Column config

```tsx
const columns: ColumnType<User>[] = [
  {
    key: "name",
    label: "Tên",
    settingLabel: "Tên nhân viên",  // tên trong modal settings
    canHide: false,                  // không cho ẩn
    canChangePosition: false,        // không cho đổi vị trí
    readOnly: true,                  // disable checkbox "Cho phép sửa"
    width: 200,
  },
];
```

### DO / DON'T

DO: `children` phải là render function — không truyền ReactNode thông thường.

DO: Truyền `onResizeColumn` và `onResizeWidth` từ render props xuống Table khi bật `columnResizable`.

DON'T: Tự quản lý state columns/pinnedColumns/showingColumns riêng lẻ — dùng `useTableSettings` để persist localStorage.

DON'T: Dùng `storageKey` trùng nhau cho 2 bảng — sẽ ghi đè settings lẫn nhau.

DON'T: Dùng `useEffect` để gọi API sau khi settings thay đổi — dùng `onConfirm` callback.
