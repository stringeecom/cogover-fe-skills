---
title: Cách sử dụng TableSettings Component của ui-kit
impact: HIGH
impactDescription: Sử dụng TableSettings sai dẫn đến mất đồng bộ giữa cài đặt cột và Table, không persist được cấu hình, kéo thả cột không hoạt động, và trải nghiệm người dùng kém
tags: table-settings, columns, pinned-columns, showing-columns, editable-columns, drag-drop, useTableSettings, ui-kit
---

## Cách sử dụng TableSettings Component của ui-kit

Đường dẫn component: `ui-kit/src/components/TableSettings`
Hook quản lý state: `ui-kit/src/components/TableSettings/hooks/useTableSettings`

Props bắt buộc: `dataKey`, `settings`, `onChangeSettings`, `children`.

Props tùy chọn: `text`, `disablePinnedColumn`, `showEditableSelect`, `renderSubColumnsSelectButton`, `onConfirm`, `ref`.

---

### RULE-TABLE-SETTINGS-01: Luôn truyền `children` dưới dạng render function nhận các callback từ TableSettings

`children` là render function nhận object `{ showSettings, onChangeSelect, onChangeSelectAll, onResizeColumn, onResizeWidth }`. Các callback này phải được truyền xuống Table component tương ứng để đồng bộ state giữa TableSettings và Table.

**Sai:**

```tsx
// ❌ Truyền children dạng ReactNode thông thường — không nhận được các callback
<TableSettings
  dataKey="id"
  settings={{ columns, showingColumns, pinnedColumns }}
  onChangeSettings={{ columns: setColumns, showingColumns: setShowingColumns, pinnedColumns: setPinnedColumns }}
>
  <Table dataSource={data} dataKey="id" columns={columns} />
</TableSettings>
```

**Đúng:**

```tsx
// ✅ Truyền children dạng render function và sử dụng các callback
<TableSettings
  dataKey="id"
  settings={{ columns, showingColumns, pinnedColumns }}
  onChangeSettings={{
    columns: setColumns,
    showingColumns: setShowingColumns,
    pinnedColumns: setPinnedColumns,
    selectedItems: setSelectedItems,
  }}
>
  {({ showSettings, onChangeSelect, onChangeSelectAll, onResizeColumn, onResizeWidth }) => (
    <Table
      dataSource={data}
      dataKey="id"
      columns={columns}
      showingColumns={showingColumns}
      pinnedColumns={pinnedColumns}
      selectable
      selectedItems={selectedItems}
      columnResizable
      onChangeSelect={onChangeSelect}
      onChangeSelectAll={onChangeSelectAll}
      onResizeColumn={onResizeColumn}
      onResizeWidth={onResizeWidth}
    />
  )}
</TableSettings>
```

---

### RULE-TABLE-SETTINGS-02: Sử dụng `useTableSettings` hook để quản lý state và tự động persist vào localStorage

`useTableSettings` quản lý `columns`, `pinnedColumns`, `showingColumns`, `sortBy`, `sortDir` và tự động lưu vào localStorage qua `storageKey`. Không tự quản lý state riêng lẻ khi cần persist.

**Sai:**

```tsx
// ❌ Tự quản lý state riêng lẻ — mất khả năng persist khi reload
const [columns, setColumns] = useState(initColumns);
const [showingColumns, setShowingColumns] = useState(initColumns.map((c) => c.key));
const [pinnedColumns, setPinnedColumns] = useState({ left: [], right: [] });
const [sortBy, setSortBy] = useState<string | null>(null);
const [sortDir, setSortDir] = useState<"asc" | "desc" | null>(null);
```

**Đúng:**

```tsx
// ✅ Dùng useTableSettings để quản lý tập trung và tự động persist
const {
  columns,
  setColumns,
  showingColumns,
  setShowingColumns,
  pinnedColumns,
  setPinnedColumns,
  sortBy,
  setSortBy,
  sortDir,
  setSortDir,
} = useTableSettings<RecordItem>({
  storageKey: "user-list-settings",
  initState: {
    initColumns,
    initPinnedColumns: { left: ["name"], right: [] },
    initShowingColumns: ["name", "email", "status"],
    initSortBy: "name",
    initSortDir: "asc",
  },
});
```

---

### RULE-TABLE-SETTINGS-03: Truyền `storageKey` là chuỗi duy nhất cho mỗi bảng khi dùng `useTableSettings`

`storageKey` được dùng để tạo key trong localStorage theo format `table-setting-{storageKey}`. Nếu hai bảng dùng cùng `storageKey`, cài đặt sẽ bị ghi đè lẫn nhau.

**Sai:**

```tsx
// ❌ Dùng storageKey trùng nhau cho hai bảng khác nhau
const userTableSettings = useTableSettings<User>({
  storageKey: "table",
  initState: { initColumns: userColumns },
});

const orderTableSettings = useTableSettings<Order>({
  storageKey: "table",
  initState: { initColumns: orderColumns },
});
```

**Đúng:**

```tsx
// ✅ Dùng storageKey riêng biệt cho từng bảng
const userTableSettings = useTableSettings<User>({
  storageKey: "user-list",
  initState: { initColumns: userColumns },
});

const orderTableSettings = useTableSettings<Order>({
  storageKey: "order-list",
  initState: { initColumns: orderColumns },
});
```

---

### RULE-TABLE-SETTINGS-04: Truyền `settings` và `onChangeSettings` đầy đủ và đồng bộ với nhau

`settings` chứa giá trị hiện tại (`columns`, `showingColumns`, `pinnedColumns`, `editableColumns`). `onChangeSettings` chứa các setter tương ứng. Phải đảm bảo mỗi key trong `settings` có setter tương ứng trong `onChangeSettings`.

**Sai:**

```tsx
// ❌ Thiếu setter tương ứng — thay đổi showingColumns sẽ không được cập nhật
<TableSettings
  dataKey="id"
  settings={{ columns, showingColumns, pinnedColumns }}
  onChangeSettings={{
    columns: setColumns,
    // Thiếu showingColumns và pinnedColumns setter
  }}
>
  {({ showSettings }) => <button onClick={showSettings}>Cài đặt</button>}
</TableSettings>
```

**Đúng:**

```tsx
// ✅ Truyền đầy đủ settings và onChangeSettings tương ứng
<TableSettings
  dataKey="id"
  settings={{
    columns,
    showingColumns,
    pinnedColumns,
    editableColumns,
  }}
  onChangeSettings={{
    columns: setColumns,
    showingColumns: setShowingColumns,
    pinnedColumns: setPinnedColumns,
    editableColumns: setEditableColumns,
    selectedItems: setSelectedItems,
  }}
>
  {({ showSettings, onChangeSelect, onChangeSelectAll, onResizeColumn, onResizeWidth }) => (
    <Table
      dataSource={data}
      dataKey="id"
      columns={columns}
      showingColumns={showingColumns}
      pinnedColumns={pinnedColumns}
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

---

### RULE-TABLE-SETTINGS-05: Dùng `disablePinnedColumn` khi không cần tính năng ghim cột trái/phải

Khi `disablePinnedColumn={true}`, modal cài đặt sẽ ẩn các nhóm "Cố định bên trái" và "Cố định bên phải", chỉ hiển thị danh sách "Cột thông thường". Nút di chuyển cột giữa các vùng cũng bị ẩn.

**Sai:**

```tsx
// ❌ Truyền pinnedColumns rỗng nhưng vẫn hiển thị UI ghim cột — gây nhầm lẫn
<TableSettings
  dataKey="id"
  settings={{ columns, showingColumns, pinnedColumns: { left: [], right: [] } }}
  onChangeSettings={{
    columns: setColumns,
    showingColumns: setShowingColumns,
    pinnedColumns: setPinnedColumns,
  }}
>
  {({ showSettings }) => <button onClick={showSettings}>Cài đặt</button>}
</TableSettings>
```

**Đúng:**

```tsx
// ✅ Dùng disablePinnedColumn khi không cần ghim cột
<TableSettings
  dataKey="id"
  disablePinnedColumn
  settings={{ columns, showingColumns }}
  onChangeSettings={{
    columns: setColumns,
    showingColumns: setShowingColumns,
  }}
>
  {({ showSettings }) => <button onClick={showSettings}>Cài đặt</button>}
</TableSettings>
```

---

### RULE-TABLE-SETTINGS-06: Sử dụng `text` prop để tùy chỉnh nhãn hiển thị trong modal cài đặt

`text` cho phép tùy chỉnh tiêu đề, placeholder tìm kiếm, nhãn các nhóm cột, nhãn nút footer, và helper text. Không override bằng CSS hay tạo modal riêng.

**Sai:**

```tsx
// ❌ Tự tạo modal cài đặt riêng — mất tính năng drag-drop và đồng bộ
<Modal title="Cài đặt cột" visible={showSettings}>
  {columns.map((col) => (
    <Checkbox
      key={col.key}
      checked={showingColumns.includes(col.key)}
      onChange={() => toggleColumn(col.key)}
    >
      {col.label}
    </Checkbox>
  ))}
</Modal>
```

**Đúng:**

```tsx
// ✅ Dùng text prop để tùy chỉnh nhãn
<TableSettings
  dataKey="id"
  text={{
    title: "Tùy chỉnh cột hiển thị",
    searchPlaceholder: "Tìm kiếm cột...",
    pinnedLeftColumnTitle: "Ghim bên trái",
    pinnedMiddleColumnTitle: "Cột bình thường",
    pinnedRightColumnTitle: "Ghim bên phải",
    confirmLabel: "Xác nhận",
    cancelLabel: "Hủy bỏ",
    helper: "Kéo thả để sắp xếp thứ tự cột",
  }}
  settings={{ columns, showingColumns, pinnedColumns }}
  onChangeSettings={{
    columns: setColumns,
    showingColumns: setShowingColumns,
    pinnedColumns: setPinnedColumns,
  }}
>
  {({ showSettings }) => <button onClick={showSettings}>Cài đặt</button>}
</TableSettings>
```

---

### RULE-TABLE-SETTINGS-07: Sử dụng `onConfirm` để xử lý logic sau khi người dùng nhấn Lưu (ví dụ: gọi API lưu cài đặt)

`onConfirm` được gọi khi người dùng nhấn nút Lưu hoặc khi resize cột. Callback nhận `{ columns, showingColumns, pinnedColumns, editableColumns }`. Sử dụng callback này để gọi API lưu cài đặt lên server.

**Sai:**

```tsx
// ❌ Theo dõi state thay đổi bằng useEffect để gọi API — dễ gọi thừa
useEffect(() => {
  saveSettingsToServer({ columns, showingColumns, pinnedColumns });
}, [columns, showingColumns, pinnedColumns]);
```

**Đúng:**

```tsx
// ✅ Dùng onConfirm để gọi API khi người dùng xác nhận
<TableSettings
  dataKey="id"
  settings={{ columns, showingColumns, pinnedColumns }}
  onChangeSettings={{
    columns: setColumns,
    showingColumns: setShowingColumns,
    pinnedColumns: setPinnedColumns,
  }}
  onConfirm={(settings) => {
    saveSettingsToServer({
      columns: settings.columns.map((col) => ({ key: col.key, width: col.width })),
      showingColumns: settings.showingColumns,
      pinnedColumns: settings.pinnedColumns,
    });
  }}
>
  {({ showSettings }) => <button onClick={showSettings}>Cài đặt</button>}
</TableSettings>
```

---

### RULE-TABLE-SETTINGS-08: Sử dụng `ref` và phương thức `reset` để reset cài đặt về trạng thái ban đầu

`TableSettingsRef` cung cấp method `reset(params?)` để reset lại `columns`, `showingColumns`, `pinnedColumns`, `editableColumns`. Nếu không truyền params, sẽ reset về giá trị ban đầu từ props `settings`.

**Sai:**

```tsx
// ❌ Tự reset từng state riêng lẻ — dễ bỏ sót và mất đồng bộ
const handleReset = () => {
  setColumns(initColumns);
  setShowingColumns(initColumns.map((c) => c.key));
  // Quên reset pinnedColumns
};
```

**Đúng:**

```tsx
// ✅ Dùng ref.reset() để reset tất cả cùng lúc
const tableSettingsRef = useRef<TableSettingsRef>(null);

const handleReset = () => {
  tableSettingsRef.current?.reset();
};

// Hoặc reset với giá trị tùy chỉnh
const handleResetCustom = () => {
  tableSettingsRef.current?.reset({
    columns: newColumns,
    showingColumns: newColumns.map((c) => c.key),
    pinnedColumns: { left: [], right: [] },
    editableColumns: [],
  });
};

<TableSettings
  ref={tableSettingsRef}
  dataKey="id"
  settings={{ columns, showingColumns, pinnedColumns }}
  onChangeSettings={{
    columns: setColumns,
    showingColumns: setShowingColumns,
    pinnedColumns: setPinnedColumns,
  }}
>
  {({ showSettings }) => (
    <div>
      <button onClick={showSettings}>Cài đặt</button>
      <button onClick={handleReset}>Khôi phục mặc định</button>
    </div>
  )}
</TableSettings>
```

---

### RULE-TABLE-SETTINGS-09: Bật `showEditableSelect` khi cần cho phép cấu hình cột có thể chỉnh sửa trực tiếp

Khi `showEditableSelect={true}`, mỗi cột trong modal cài đặt sẽ hiển thị thêm checkbox "Cho phép sửa" bên cạnh. Cần truyền `editableColumns` trong `settings` và `onChangeSettings` tương ứng.

**Sai:**

```tsx
// ❌ Bật showEditableSelect nhưng không truyền editableColumns — checkbox sẽ không hoạt động đúng
<TableSettings
  dataKey="id"
  showEditableSelect
  settings={{ columns, showingColumns, pinnedColumns }}
  onChangeSettings={{
    columns: setColumns,
    showingColumns: setShowingColumns,
    pinnedColumns: setPinnedColumns,
  }}
>
  {({ showSettings }) => <button onClick={showSettings}>Cài đặt</button>}
</TableSettings>
```

**Đúng:**

```tsx
// ✅ Truyền đầy đủ editableColumns khi bật showEditableSelect
<TableSettings
  dataKey="id"
  showEditableSelect
  settings={{
    columns,
    showingColumns,
    pinnedColumns,
    editableColumns,
  }}
  onChangeSettings={{
    columns: setColumns,
    showingColumns: setShowingColumns,
    pinnedColumns: setPinnedColumns,
    editableColumns: setEditableColumns,
  }}
>
  {({ showSettings }) => <button onClick={showSettings}>Cài đặt</button>}
</TableSettings>
```

---

### RULE-TABLE-SETTINGS-10: Sử dụng `column.canHide` và `column.canChangePosition` để kiểm soát hành vi từng cột trong modal

`canHide: false` sẽ disable checkbox ẩn/hiện của cột đó (luôn hiển thị). `canChangePosition: false` sẽ ẩn nút di chuyển vị trí và disable drag-drop cho cột đó. Sử dụng `column.settingLabel` để đặt tên hiển thị riêng trong modal.

**Sai:**

```tsx
// ❌ Lọc cột khỏi settings để ngăn ẩn — cột sẽ không xuất hiện trong danh sách cài đặt
const settingsColumns = columns.filter((col) => col.key !== "name");

<TableSettings
  dataKey="id"
  settings={{ columns: settingsColumns, showingColumns, pinnedColumns }}
  onChangeSettings={{
    columns: setColumns,
    showingColumns: setShowingColumns,
    pinnedColumns: setPinnedColumns,
  }}
>
  {({ showSettings }) => <button onClick={showSettings}>Cài đặt</button>}
</TableSettings>
```

**Đúng:**

```tsx
// ✅ Dùng canHide và canChangePosition trên cấu hình cột
const columns: ColumnType<User>[] = [
  {
    key: "name",
    label: "Tên",
    settingLabel: "Tên nhân viên",
    canHide: false,
    canChangePosition: false,
    width: 200,
  },
  {
    key: "email",
    label: "Email",
    width: 300,
  },
  {
    key: "status",
    label: "Trạng thái",
    settingLabel: "Trạng thái hoạt động",
    width: 150,
  },
];
```

---

### RULE-TABLE-SETTINGS-11: Sử dụng `text.defaultActionFooter` để thêm nội dung tùy chỉnh vào footer bên trái

`defaultActionFooter` cho phép render nội dung tùy chỉnh ở phía bên trái của footer (bên trái các nút Lưu/Hủy). Thường dùng để thêm nút "Khôi phục mặc định".

**Sai:**

```tsx
// ❌ Tự render footer bên ngoài TableSettings — mất đồng bộ với modal
<TableSettings
  dataKey="id"
  settings={{ columns, showingColumns, pinnedColumns }}
  onChangeSettings={{
    columns: setColumns,
    showingColumns: setShowingColumns,
    pinnedColumns: setPinnedColumns,
  }}
>
  {({ showSettings }) => <button onClick={showSettings}>Cài đặt</button>}
</TableSettings>
<div className={cx("flex justify-between")}>
  <button onClick={handleReset}>Khôi phục mặc định</button>
</div>
```

**Đúng:**

```tsx
// ✅ Dùng text.defaultActionFooter để render nội dung footer tùy chỉnh
<TableSettings
  dataKey="id"
  text={{
    defaultActionFooter: (
      <Button variant="text" onClick={handleResetToDefault}>
        Khôi phục mặc định
      </Button>
    ),
  }}
  settings={{ columns, showingColumns, pinnedColumns }}
  onChangeSettings={{
    columns: setColumns,
    showingColumns: setShowingColumns,
    pinnedColumns: setPinnedColumns,
  }}
>
  {({ showSettings }) => <button onClick={showSettings}>Cài đặt</button>}
</TableSettings>
```

---

### RULE-TABLE-SETTINGS-12: Truyền callback `onResizeColumn` và `onResizeWidth` từ children params xuống Table khi bật `columnResizable`

Khi Table có `columnResizable`, phải truyền `onResizeColumn` và `onResizeWidth` từ render function params để TableSettings có thể đồng bộ chiều rộng cột khi resize. Nếu thiếu, chiều rộng cột sẽ không được lưu lại.

**Sai:**

```tsx
// ❌ Thiếu onResizeColumn và onResizeWidth — resize cột sẽ không được lưu
<TableSettings
  dataKey="id"
  settings={{ columns, showingColumns, pinnedColumns }}
  onChangeSettings={{ columns: setColumns, showingColumns: setShowingColumns, pinnedColumns: setPinnedColumns }}
>
  {({ showSettings, onChangeSelect, onChangeSelectAll }) => (
    <Table
      dataSource={data}
      dataKey="id"
      columns={columns}
      columnResizable
      onChangeSelect={onChangeSelect}
      onChangeSelectAll={onChangeSelectAll}
    />
  )}
</TableSettings>
```

**Đúng:**

```tsx
// ✅ Truyền đầy đủ onResizeColumn và onResizeWidth khi bật columnResizable
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
      columnResizable
      showingColumns={showingColumns}
      pinnedColumns={pinnedColumns}
      onChangeSelect={onChangeSelect}
      onChangeSelectAll={onChangeSelectAll}
      onResizeColumn={onResizeColumn}
      onResizeWidth={onResizeWidth}
    />
  )}
</TableSettings>
```

---

### RULE-TABLE-SETTINGS-13: Sử dụng `renderSubColumnsSelectButton` để thêm nút chọn sub-columns cho các cột lookup

`renderSubColumnsSelectButton` nhận params gồm `column`, `columns`, `showingColumns`, `pinnedColumns`, cùng các hàm `changeColumns`, `changeShowingColumns`, `changePinnedColumns`. Dùng để render nút mở rộng cho cột có quan hệ lookup.

**Sai:**

```tsx
// ❌ Tự render nút sub-columns bên ngoài TableSettings — không đồng bộ state
<TableSettings
  dataKey="id"
  settings={{ columns, showingColumns, pinnedColumns }}
  onChangeSettings={{ columns: setColumns, showingColumns: setShowingColumns, pinnedColumns: setPinnedColumns }}
>
  {({ showSettings }) => <button onClick={showSettings}>Cài đặt</button>}
</TableSettings>
<SubColumnSelector columns={columns} onChange={setColumns} />
```

**Đúng:**

```tsx
// ✅ Dùng renderSubColumnsSelectButton để render nút bên trong modal cài đặt
<TableSettings
  dataKey="id"
  settings={{ columns, showingColumns, pinnedColumns }}
  onChangeSettings={{ columns: setColumns, showingColumns: setShowingColumns, pinnedColumns: setPinnedColumns }}
  renderSubColumnsSelectButton={({
    column,
    columns,
    pinnedColumns,
    showingColumns,
    changeColumns,
    changePinnedColumns,
    changeShowingColumns,
  }) => {
    return (
      <SelectLookupObjectFieldButton
        parentObjectSlug={objectSlug}
        parentColumnKey={column.key}
        columns={columns}
        onChangeColumns={changeColumns}
        showingColumns={showingColumns}
        onChangeShowingColumns={changeShowingColumns}
        pinnedColumns={pinnedColumns}
        onChangePinnedColumns={changePinnedColumns}
      />
    );
  }}
>
  {({ showSettings }) => <button onClick={showSettings}>Cài đặt</button>}
</TableSettings>
```

---

### RULE-TABLE-SETTINGS-14: Dùng `column.readOnly` để disable checkbox "Cho phép sửa" cho cột không thể chỉnh sửa

Khi `showEditableSelect` được bật, checkbox "Cho phép sửa" của các cột có `readOnly: true` sẽ bị disabled. Cột con (sub-column có key chứa dấu `.`) cũng tự động bị disabled.

**Sai:**

```tsx
// ❌ Lọc cột ra khỏi editableColumns bằng logic bên ngoài — thiếu phản hồi UI
const handleEditableChange = (checked: boolean, key: string) => {
  if (readOnlyKeys.includes(key)) return; // không có phản hồi UI
  setEditableColumns((prev) => (checked ? [...prev, key] : prev.filter((k) => k !== key)));
};
```

**Đúng:**

```tsx
// ✅ Dùng readOnly trên cấu hình cột — checkbox sẽ bị disabled và hiển thị rõ ràng
const columns: ColumnType<User>[] = [
  {
    key: "id",
    label: "ID",
    readOnly: true,
    width: 100,
  },
  {
    key: "name",
    label: "Tên",
    width: 200,
  },
  {
    key: "created_at",
    label: "Ngày tạo",
    readOnly: true,
    width: 150,
  },
];
```

---

## Tham chiếu cấu hình mặc định

| Prop | Giá trị mặc định | Mô tả |
|------|-------------------|-------|
| `text.title` | `"Tuỳ chỉnh cột dữ liệu"` | Tiêu đề modal cài đặt |
| `text.searchPlaceholder` | `"Tìm kiếm cột"` | Placeholder ô tìm kiếm |
| `text.pinnedLeftColumnTitle` | `"Cố định bên trái"` | Tiêu đề nhóm cột ghim trái |
| `text.pinnedMiddleColumnTitle` | `"Cột thông thường"` | Tiêu đề nhóm cột bình thường |
| `text.pinnedRightColumnTitle` | `"Cố định bên phải"` | Tiêu đề nhóm cột ghim phải |
| `text.confirmLabel` | `"Lưu"` | Nhãn nút xác nhận |
| `text.cancelLabel` | `"Huỷ"` | Nhãn nút hủy |
| `disablePinnedColumn` | `false` | Ẩn tính năng ghim cột trái/phải |
| `showEditableSelect` | `false` | Hiển thị checkbox "Cho phép sửa" |

## Tham chiếu useTableSettings Props

| Prop | Kiểu | Mô tả |
|------|------|-------|
| `storageKey` | `string` | Key duy nhất dùng persist vào localStorage |
| `initState.initColumns` | `ColumnType<T>[]` | Danh sách cột ban đầu |
| `initState.initShowingColumns` | `string[]` | Danh sách key cột hiển thị ban đầu (mặc định: tất cả) |
| `initState.initPinnedColumns` | `TablePinnedColumns` | Cấu hình ghim cột ban đầu (mặc định: `{ left: [], right: [] }`) |
| `initState.initSortBy` | `string \| null` | Cột sắp xếp ban đầu |
| `initState.initSortDir` | `"asc" \| "desc" \| null` | Hướng sắp xếp ban đầu |

## Tham chiếu useTableSettings Return

| Giá trị | Kiểu | Mô tả |
|---------|------|-------|
| `columns` | `ColumnType<T>[]` | Danh sách cột hiện tại (đã merge với persisted data) |
| `setColumns` | `Dispatch<SetStateAction<ColumnType<T>[]>>` | Setter cho columns |
| `showingColumns` | `string[]` | Danh sách key cột đang hiển thị |
| `setShowingColumns` | `Dispatch<SetStateAction<string[]>>` | Setter cho showingColumns |
| `pinnedColumns` | `TablePinnedColumns` | Cấu hình ghim cột hiện tại |
| `setPinnedColumns` | `Dispatch<SetStateAction<TablePinnedColumns>>` | Setter cho pinnedColumns |
| `sortBy` | `string \| null \| undefined` | Cột đang sắp xếp |
| `setSortBy` | `Dispatch<SetStateAction<...>>` | Setter cho sortBy |
| `sortDir` | `"asc" \| "desc" \| null \| undefined` | Hướng sắp xếp hiện tại |
| `setSortDir` | `Dispatch<SetStateAction<...>>` | Setter cho sortDir |
