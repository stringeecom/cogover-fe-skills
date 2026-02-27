## Cách sử dụng ListViewTable Component của ui-kit

Path: `ui-kit/src/components/ListViewTable`

Wrapper gồm `ListViewHeader` (title, action buttons) + `Table`. Kế thừa tất cả props của `Table` (trừ `className` đổi thành `tableClassName`).

### Props riêng

| Prop | Mặc định | Mô tả |
|---|---|---|
| `title` | `""` | `ReactNode` — tiêu đề (`prose-h6 text-typo-primary`) |
| `subTitle` | `""` | `ReactNode` — phụ đề (`prose-body2 text-typo-secondary`) |
| `renderActionButtons` | `() => null` | `() => ReactNode` — nút hành động (bên phải) |
| `renderContextButtons` | `() => null` | `() => ReactNode` — nút ngữ cảnh (cạnh action) |
| `renderHeadingOptions` | `undefined` | `() => ReactNode` — nội dung giữa header và bảng |
| `className` | — | Class cho container ngoài (div bao header + table) |
| `tableClassName` | — | Class cho Table bên trong |
| `fullHeight` | `true` | Thêm `h-full` cho container |

### Basic usage

```tsx
<ListViewTable
  title="Danh sách người dùng"
  subTitle={`Tổng: ${meta?.total ?? 0}`}
  dataKey="id"
  columns={columns}
  dataSource={data}
  loading={isLoading}
  selectable
  selectedItems={selectedItems}
  onChangeSelect={handleChangeSelect}
  onChangeSelectAll={handleChangeSelectAll}
  sortBy={sortBy}
  sortDir={sortDir}
  onSortChange={handleSortChange}
  pagination={{
    total: meta?.total ?? 0,
    currentPage: page,
    perPage: limit,
    onNextPage: () => setPage((p) => p + 1),
    onPrevPage: () => setPage((p) => p - 1),
    onChangePerPage: (perPage) => { setLimit(perPage); setPage(1); },
  }}
  renderActionButtons={() => (
    <div className={cx("flex gap-[1rem]")}>
      <Button startIcon={<FontAwesomeIcon icon={faPlus} />}>Thêm mới</Button>
    </div>
  )}
  renderNoData={() => <h5 className={cx("prose-h5 text-typo-secondary", "mt-[2.5rem]")}>Không có dữ liệu</h5>}
/>
```

### Kết hợp với TableSettings

```tsx
<TableSettings
  dataKey="id"
  settings={{ columns, showingColumns, pinnedColumns }}
  onChangeSettings={{ columns: setColumns, showingColumns: setShowingColumns, pinnedColumns: setPinnedColumns, selectedItems: setSelectedItems }}
>
  {({ showSettings, onChangeSelect, onChangeSelectAll, onResizeColumn, onResizeWidth }) => (
    <ListViewTable
      title="Danh sách"
      dataKey="id"
      columns={columns}
      dataSource={data}
      selectable
      columnResizable
      selectedItems={selectedItems}
      pinnedColumns={pinnedColumns}
      showingColumns={showingColumns}
      onChangeSelect={onChangeSelect}
      onChangeSelectAll={onChangeSelectAll}
      onResizeColumn={onResizeColumn}
      onResizeWidth={onResizeWidth}
      renderActionButtons={() => (
        <Button startIcon={<FontAwesomeIcon icon={faSliders} />} variant="gray" onClick={showSettings}>
          Cài đặt
        </Button>
      )}
    />
  )}
</TableSettings>
```

### DO / DON'T

DO: Phân biệt `className` (container ngoài) và `tableClassName` (Table bên trong).

DO: Dùng `renderActionButtons` và `renderContextButtons` — không tự render nút bên ngoài ListViewTable.

DO: Dùng `title` và `subTitle` — không tự tạo heading bên ngoài.

DO: Truyền tất cả Table props (`pagination`, `selectable`, `sortBy`...) trực tiếp vào ListViewTable — không tạo Table riêng bên trong.

DON'T: Tự thêm `h-full` qua `className` — `fullHeight` mặc định `true` đã xử lý.

DON'T: Tự render filter bar/search bên ngoài và thêm margin thủ công — dùng `renderHeadingOptions`.
