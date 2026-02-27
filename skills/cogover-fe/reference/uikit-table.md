---
title: Cách sử dụng Table Component của ui-kit
impact: HIGH
impactDescription: Sử dụng Table sai dẫn đến lỗi hiển thị dữ liệu, pagination không hoạt động, selection bị sai, cột bị ghim sai vị trí và hiệu năng kém
tags: table, pagination, selectable, pinned-columns, sort, group, virtualized, column-resize, ui-kit
---

## Cách sử dụng Table Component của ui-kit

Đường dẫn component: `ui-kit/src/components/Table`
Hook quản lý state: `ui-kit/src/components/Table/hooks/useTableV2`

Props chính: `dataSource`, `dataKey`, `columns`, `selectable`, `selectedItems`, `singleSelect`, `pagination`, `pinnedColumns`, `showingColumns`, `sortBy`, `sortDir`, `border`, `loading`, `fullHeight`, `virtualized`, `canReorderRow`, `columnResizable`, `renderGroup`, `renderSummaryGroup`, `paramsForRenderCell`, `emptyRows`, `renderNoData`, `className`.

Callbacks: `onChangeSelect`, `onChangeSelectAll`, `onSortChange`, `onResizeColumn`, `onDragEnd`, `onResizeWidth`, `onToggleExpandGroupKeys`.

---

### RULE-TABLE-01: Luôn truyền `dataKey` là đường dẫn đến trường unique identifier của mỗi item

`dataKey` được dùng để so sánh và xác định các item (selection, drag-and-drop, virtualization). Nó sử dụng `object-path` nên hỗ trợ nested path. Nếu thiếu hoặc sai, selection và nhiều tính năng khác sẽ không hoạt động đúng.

**Sai:**

```tsx
// ❌ Thiếu dataKey — bắt buộc phải có
<Table dataSource={users} columns={columns} />

// ❌ dataKey trỏ đến field không unique
<Table dataSource={users} dataKey="name" columns={columns} />
```

**Đúng:**

```tsx
// ✅ dataKey trỏ đến field unique identifier
<Table dataSource={users} dataKey="id" columns={columns} />

// ✅ Hỗ trợ nested path qua object-path
<Table dataSource={users} dataKey="user.id" columns={columns} />
```

---

### RULE-TABLE-02: Sử dụng `column.render` để tùy chỉnh nội dung cell, không dùng `column.label`

`column.label` là tiêu đề hiển thị ở header. `column.render` là hàm nhận `(record, index, dataSource, params)` để render nội dung cell. Nếu không truyền `render`, Table sẽ tự động lấy giá trị từ `record[column.key]` bằng `object-path`.

**Sai:**

```tsx
// ❌ Cố gắng render nội dung cell thông qua label
const columns = [
  {
    key: "status",
    label: <Badge status={item.status} />,
  },
];
```

**Đúng:**

```tsx
// ✅ Dùng render để tùy chỉnh nội dung cell
const columns: ColumnType<User>[] = [
  {
    key: "status",
    label: "Trạng thái",
    render: (record) => <Badge status={record.status} />,
  },
];

// ✅ Nếu không truyền render, Table sẽ tự lấy giá trị record["name"]
const columns: ColumnType<User>[] = [
  {
    key: "name",
    label: "Tên",
    width: 200,
  },
];
```

---

### RULE-TABLE-03: Truyền `width` cho cột bằng số (đơn vị pixel), không dùng chuỗi

`column.width` nhận kiểu `number` (pixel). Cột cuối cùng tự động co giãn để lấp đầy không gian còn lại nhờ `flex-1`. Có thể truyền thêm `minWidth` để giới hạn chiều rộng tối thiểu khi resize.

**Sai:**

```tsx
// ❌ width không nhận chuỗi
const columns = [
  { key: "name", label: "Tên", width: "200px" },
];
```

**Đúng:**

```tsx
// ✅ width là number (pixel)
const columns: ColumnType<User>[] = [
  { key: "name", label: "Tên", width: 200, minWidth: 100 },
  { key: "email", label: "Email", width: 300 },
];
```

---

### RULE-TABLE-04: Sử dụng `selectable` kết hợp `selectedItems`, `onChangeSelect` và `onChangeSelectAll` cho chức năng chọn hàng

Không tự render checkbox trong cột. Table đã tích hợp sẵn cột checkbox khi bật `selectable`. Dùng `singleSelect` nếu chỉ cho phép chọn một hàng (hiển thị Radio thay vì Checkbox).

**Sai:**

```tsx
// ❌ Tự render checkbox trong column — dư thừa và gây conflict
const columns = [
  {
    key: "select",
    label: "",
    render: (record) => (
      <Checkbox checked={selected.includes(record.id)} onChange={() => toggle(record)} />
    ),
  },
  // ...
];
<Table dataSource={data} dataKey="id" columns={columns} />
```

**Đúng:**

```tsx
// ✅ Sử dụng props selectable tích hợp sẵn
<Table
  dataSource={data}
  dataKey="id"
  columns={columns}
  selectable
  selectedItems={selectedItems}
  onChangeSelect={(checked, item) => {
    // xử lý chọn/bỏ chọn một item
  }}
  onChangeSelectAll={(checked, items) => {
    // xử lý chọn/bỏ chọn tất cả
  }}
/>

// ✅ Chọn đơn với Radio
<Table
  dataSource={data}
  dataKey="id"
  columns={columns}
  selectable
  singleSelect
  selectedItems={selectedItems}
  onChangeSelect={(checked, item) => {
    setSelectedItems([item]);
  }}
/>
```

---

### RULE-TABLE-05: Sử dụng `disabledSelect` và `disabledSelectTooltip` để vô hiệu hóa chọn hàng có điều kiện

`disabledSelect` là hàm `(record, index, dataSource, params) => boolean`. Khi trả về `true`, checkbox/radio của hàng đó sẽ bị disabled. Kết hợp `disabledSelectTooltip` để hiển thị lý do.

**Sai:**

```tsx
// ❌ Lọc dataSource để ẩn hàng không được chọn
<Table
  dataSource={data.filter((item) => item.canSelect)}
  dataKey="id"
  columns={columns}
  selectable
/>
```

**Đúng:**

```tsx
// ✅ Dùng disabledSelect để disable chọn có điều kiện
<Table
  dataSource={data}
  dataKey="id"
  columns={columns}
  selectable
  selectedItems={selectedItems}
  disabledSelect={(record) => !record.canSelect}
  disabledSelectTooltip={(record) => `Không thể chọn: ${record.reason}`}
  onChangeSelect={handleChangeSelect}
  onChangeSelectAll={handleChangeSelectAll}
/>
```

---

### RULE-TABLE-06: Sử dụng `pinnedColumns` để ghim cột trái/phải, không tự tạo hai bảng riêng

`pinnedColumns` nhận object `{ left?: string[], right?: string[] }` chứa các `column.key` cần ghim. Table sẽ tự chia cột thành 3 vùng (left, middle, right) với shadow và scroll đồng bộ.

**Sai:**

```tsx
// ❌ Tự tạo hai bảng riêng cho cột ghim
<div className="flex">
  <Table dataSource={data} dataKey="id" columns={fixedColumns} />
  <Table dataSource={data} dataKey="id" columns={scrollColumns} />
</div>
```

**Đúng:**

```tsx
// ✅ Dùng pinnedColumns
<Table
  dataSource={data}
  dataKey="id"
  columns={columns}
  pinnedColumns={{
    left: ["name"],
    right: ["actions"],
  }}
/>
```

---

### RULE-TABLE-07: Không sử dụng `canReorderRow` cùng với `pinnedColumns` hoặc grouped data

`canReorderRow` sẽ throw Error nếu kết hợp với pinned columns hoặc khi `dataSource` là object (grouped table). Đây là ràng buộc kỹ thuật của component.

**Sai:**

```tsx
// ❌ Sẽ throw Error tại runtime
<Table
  dataSource={data}
  dataKey="id"
  columns={columns}
  canReorderRow
  pinnedColumns={{ left: ["name"] }}
  onDragEnd={handleDragEnd}
/>
```

**Đúng:**

```tsx
// ✅ canReorderRow chỉ dùng với flat dataSource và không có pinnedColumns
<Table
  dataSource={data}
  dataKey="id"
  columns={columns}
  canReorderRow
  onDragEnd={handleDragEnd}
/>
```

---

### RULE-TABLE-08: Sử dụng `onSortChange` với `sortBy` và `sortDir` cho sắp xếp, kết hợp `column.sortable`

Đánh dấu cột có thể sắp xếp bằng `column.sortable = true`. Table sẽ hiển thị icon sort và gọi `onSortChange(sortBy, sortDir)` khi click. Chu kỳ sort: `asc` → `desc` → `null` (reset).

**Sai:**

```tsx
// ❌ Tự xử lý sort trong column label
const columns = [
  {
    key: "name",
    label: (
      <button onClick={() => handleSort("name")}>
        Tên <SortIcon />
      </button>
    ),
  },
];
```

**Đúng:**

```tsx
// ✅ Dùng sortable và onSortChange
const columns: ColumnType<User>[] = [
  { key: "name", label: "Tên", sortable: true, width: 200 },
  { key: "email", label: "Email", sortable: true, width: 300 },
];

<Table
  dataSource={data}
  dataKey="id"
  columns={columns}
  sortBy={sortBy}
  sortDir={sortDir}
  onSortChange={(newSortBy, newSortDir) => {
    setSortBy(newSortBy);
    setSortDir(newSortDir);
  }}
/>
```

---

### RULE-TABLE-09: Sử dụng prop `pagination` tích hợp, không tự render Pagination bên ngoài

Table đã tích hợp component `Pagination` bên trong. Truyền props pagination trực tiếp. Hỗ trợ thêm `alwaysShow` để luôn hiển thị pagination ngay cả khi tổng số bản ghi nhỏ hơn perPage nhỏ nhất.

**Sai:**

```tsx
// ❌ Render Pagination bên ngoài Table — mất đồng bộ scroll-to-top
<Table dataSource={data} dataKey="id" columns={columns} />
<Pagination total={100} page={page} perPage={20} onNextPage={...} onPrevPage={...} />
```

**Đúng:**

```tsx
// ✅ Dùng prop pagination tích hợp — tự động scroll-to-top khi chuyển trang
<Table
  dataSource={data}
  dataKey="id"
  columns={columns}
  pagination={{
    total: 100,
    page: page,
    perPage: 20,
    onNextPage: () => setPage((p) => p + 1),
    onPrevPage: () => setPage((p) => p - 1),
    onChangePerPage: (perPage) => {
      setPerPage(perPage);
      setPage(1);
    },
  }}
/>

// ✅ Luôn hiển thị pagination dù ít dữ liệu
<Table
  dataSource={data}
  dataKey="id"
  columns={columns}
  pagination={{
    alwaysShow: true,
    total: 5,
    page: 1,
    perPage: 20,
    onNextPage: () => {},
    onPrevPage: () => {},
    onChangePerPage: () => {},
  }}
/>
```

---

### RULE-TABLE-10: Truyền `dataSource` dạng object khi sử dụng grouped table

Khi cần nhóm dữ liệu, truyền `dataSource` dạng `Record<string | number, T[]>`. Kết hợp `renderGroup` để render header nhóm và `renderSummaryGroup` để render summary cuối nhóm. Quản lý expand/collapse qua `defaultExpandedGroupKeys`, `expandedGroupKeys` và `onToggleExpandGroupKeys`.

**Sai:**

```tsx
// ❌ Tự nhóm dữ liệu và render nhiều Table
{Object.entries(groupedData).map(([key, items]) => (
  <div key={key}>
    <h3>{key}</h3>
    <Table dataSource={items} dataKey="id" columns={columns} />
  </div>
))}
```

**Đúng:**

```tsx
// ✅ Truyền object cho dataSource
<Table
  dataSource={{
    "Nhóm A": groupA,
    "Nhóm B": groupB,
  }}
  dataKey="id"
  columns={columns}
  renderGroup={(key, dataSource, index) => (
    <span className={cx("prose-body1 text-typo-primary")}>{key} ({dataSource.length})</span>
  )}
  renderSummaryGroup={(key, dataSource, index) => (
    <span>Tổng: {dataSource.reduce((sum, item) => sum + item.amount, 0)}</span>
  )}
  defaultExpandedGroupKeys={["Nhóm A"]}
/>
```

---

### RULE-TABLE-11: Sử dụng `virtualized` cho bảng có nhiều hàng để cải thiện hiệu năng

Khi bảng có nhiều hàng (> 100), bật `virtualized` để chỉ render các hàng đang hiển thị. Cần truyền `maxHeight` để giới hạn chiều cao vùng scroll. Hỗ trợ `stickyBottomRowIndexes` cho các hàng cố định ở cuối.

**Sai:**

```tsx
// ❌ Render hàng nghìn hàng mà không dùng virtualization — gây lag
<Table dataSource={thousandItems} dataKey="id" columns={columns} />
```

**Đúng:**

```tsx
// ✅ Bật virtualized cho bảng nhiều hàng
<Table
  dataSource={thousandItems}
  dataKey="id"
  columns={columns}
  virtualized={{
    enabled: true,
    maxHeight: 600,
  }}
/>

// ✅ Với hàng sticky ở cuối (ví dụ: hàng tổng)
<Table
  dataSource={items}
  dataKey="id"
  columns={columns}
  virtualized={{
    enabled: true,
    maxHeight: 600,
    stickyBottomRowIndexes: [items.length - 1],
  }}
/>
```

---

### RULE-TABLE-12: Sử dụng `showingColumns` để ẩn/hiện cột, không lọc mảng `columns`

`showingColumns` nhận mảng các `column.key` cần hiển thị. Table sẽ tự lọc và chỉ render các cột trong danh sách này. Điều này giữ nguyên cấu hình cột gốc và cho phép toggle dễ dàng.

**Sai:**

```tsx
// ❌ Lọc mảng columns — mất cấu hình gốc khi toggle
<Table
  dataSource={data}
  dataKey="id"
  columns={columns.filter((col) => visibleKeys.includes(col.key))}
/>
```

**Đúng:**

```tsx
// ✅ Dùng showingColumns
<Table
  dataSource={data}
  dataKey="id"
  columns={columns}
  showingColumns={["name", "email", "status"]}
/>
```

---

### RULE-TABLE-13: Sử dụng `useTableV2` hook để quản lý state cho Table

`useTableV2` cung cấp quản lý state đầy đủ: columns, pinnedColumns, showingColumns, page, limit, sortBy, sortDir. Nó tự động đồng bộ với API settings và persist limit vào localStorage.

**Sai:**

```tsx
// ❌ Tự quản lý hết state — mất đồng bộ với API settings
const [columns, setColumns] = useState(initColumns);
const [page, setPage] = useState(1);
const [limit, setLimit] = useState(20);
const [sortBy, setSortBy] = useState(null);
const [sortDir, setSortDir] = useState(null);
const [showingColumns, setShowingColumns] = useState([]);
const [pinnedColumns, setPinnedColumns] = useState({});
```

**Đúng:**

```tsx
// ✅ Dùng useTableV2 để quản lý tập trung
const {
  columns,
  setColumns,
  pinnedColumns,
  setPinnedColumns,
  showingColumns,
  setShowingColumns,
  page,
  setPage,
  limit,
  setLimit,
  sortBy,
  setSortBy,
  sortDir,
  setSortDir,
} = useTableV2({
  tableName: "user-list",
  initColumns: userColumns,
  initPinnedColumns: { left: ["name"] },
  initShowingColumns: ["name", "email", "role"],
  initSortBy: "name",
  initSortDir: "asc",
  initPage: 1,
  initLimit: 20,
});
```

---

### RULE-TABLE-14: Sử dụng `border` để cấu hình đường viền, không override bằng className

`border` nhận object `{ wrapper?: boolean, horizontal?: boolean, vertical?: boolean }`. Mặc định: `wrapper: false`, `horizontal: true`, `vertical: false`.

**Sai:**

```tsx
// ❌ Override border bằng className
<Table
  dataSource={data}
  dataKey="id"
  columns={columns}
  className="border [&_td]:border-r"
/>
```

**Đúng:**

```tsx
// ✅ Dùng prop border
<Table
  dataSource={data}
  dataKey="id"
  columns={columns}
  border={{ wrapper: true, horizontal: true, vertical: true }}
/>
```

---

### RULE-TABLE-15: Sử dụng `paramsForRenderCell` để truyền dữ liệu bổ sung vào các hàm render

Khi cần truyền thêm dữ liệu (permissions, config...) vào `column.render`, `disabledSelect`, `disabledRow`, sử dụng `paramsForRenderCell`. Tham số thứ 4 của các hàm render sẽ nhận giá trị này.

**Sai:**

```tsx
// ❌ Dùng closure để truy cập biến bên ngoài — gây re-render không cần thiết
const columns: ColumnType<User>[] = [
  {
    key: "actions",
    label: "Hành động",
    render: (record) => canEdit ? <EditButton /> : null,
  },
];
```

**Đúng:**

```tsx
// ✅ Dùng paramsForRenderCell
const columns: ColumnType<User>[] = [
  {
    key: "actions",
    label: "Hành động",
    render: (record, index, dataSource, params) =>
      params.canEdit ? <EditButton /> : null,
  },
];

<Table
  dataSource={data}
  dataKey="id"
  columns={columns}
  paramsForRenderCell={{ canEdit: hasPermission("edit") }}
/>
```

---

### RULE-TABLE-16: Sử dụng `columnResizable` kết hợp `onResizeColumn` cho tính năng thay đổi chiều rộng cột

Bật `columnResizable` để cho phép kéo resize cột. Table sẽ gọi `onResizeColumn(columnKey, newWidth)` khi người dùng hoàn tất kéo. Dùng `columnResizeImmediately` (mặc định `true`) để hiển thị preview ngay khi kéo.

**Sai:**

```tsx
// ❌ Tự implement resize — phức tạp và không đồng bộ
<Table dataSource={data} dataKey="id" columns={columns} />
// ... custom resize logic bên ngoài
```

**Đúng:**

```tsx
// ✅ Dùng columnResizable tích hợp
<Table
  dataSource={data}
  dataKey="id"
  columns={columns}
  columnResizable
  onResizeColumn={(columnKey, newWidth) => {
    setColumns((prev) =>
      prev.map((col) => (col.key === columnKey ? { ...col, width: newWidth } : col)),
    );
  }}
/>
```

---

## Tham chiếu cấu hình mặc định

| Prop | Giá trị mặc định | Mô tả |
|------|-------------------|-------|
| `stickyHeader` | `true` | Header cố định khi scroll |
| `fullHeight` | `true` | Table chiếm toàn bộ chiều cao container |
| `loading` | `false` | Hiển thị skeleton loading (5 hàng) |
| `selectable` | `false` | Hiển thị cột checkbox/radio |
| `singleSelect` | `false` | Chọn đơn (Radio) thay vì đa chọn (Checkbox) |
| `columnResizable` | `false` | Cho phép kéo resize cột |
| `canReorderRow` | `false` | Cho phép kéo thả đổi thứ tự hàng |
| `columnResizeImmediately` | `true` | Preview resize ngay khi kéo |
| `border.wrapper` | `false` | Viền bao quanh table |
| `border.horizontal` | `true` | Đường kẻ ngang giữa các hàng |
| `border.vertical` | `false` | Đường kẻ dọc giữa các cột |

## Tham chiếu ColumnType

| Prop | Kiểu | Mô tả |
|------|------|-------|
| `key` | `string` | Key duy nhất, đồng thời là path lấy giá trị từ record |
| `label` | `ReactNode` | Tiêu đề hiển thị ở header |
| `renderLabel` | `() => ReactNode` | Tùy chỉnh render header (thay thế label) |
| `render` | `(record, index, dataSource, params) => ReactNode` | Tùy chỉnh render nội dung cell |
| `width` | `number` | Chiều rộng cột (pixel) |
| `minWidth` | `number` | Chiều rộng tối thiểu (pixel) |
| `align` | `"left" \| "right" \| "center"` | Căn chỉnh nội dung |
| `sortable` | `boolean` | Cho phép sắp xếp |
| `fullWidth` | `boolean` | Nội dung cell chiếm toàn bộ chiều rộng |
| `paddingX` | `number` | Padding ngang cho cell (mặc định 12px) |
| `helpText` | `ReactNode` | Tooltip hướng dẫn bên cạnh label |
| `tooltip` | `object` | Cấu hình tooltip cho cell |
| `rowSpan` | `(record, index, dataSource, params) => number` | Gộp hàng |
| `colSpan` | `(record, index, dataSource, params) => number` | Gộp cột |
| `disabled` | `(record, index, dataSource, params) => boolean` | Vô hiệu hóa cell (opacity 50%) |
| `cellAttributes` | `(record, index, dataSource, params) => HTMLAttributes` | Thuộc tính HTML tùy chỉnh cho cell |
| `styles.backgroundColor` | `string \| RowFunction` | Màu nền tùy chỉnh cho cell |
