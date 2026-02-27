---
title: Cách sử dụng ListViewTable Component của ui-kit
impact: HIGH
impactDescription: Sử dụng ListViewTable sai dẫn đến header không hiển thị đúng, bố cục bảng bị vỡ, mất tính năng heading options và action buttons, className bị ghi đè sai vị trí
tags: list-view-table, table, header, action-buttons, context-buttons, heading-options, fullHeight, ui-kit
---

## Cách sử dụng ListViewTable Component của ui-kit

Đường dẫn component: `ui-kit/src/components/ListViewTable`
Subcomponent: `ui-kit/src/components/ListViewTable/ListViewHeader`

ListViewTable là wrapper component bao gồm một **ListViewHeader** (tiêu đề, phụ đề, các nút hành động) và một **Table** bên dưới. Nó kế thừa toàn bộ props của Table (trừ `className` của Table được đổi thành `tableClassName`).

Props riêng của ListViewTable: `title`, `subTitle`, `renderActionButtons`, `renderContextButtons`, `renderHeadingOptions`, `tableClassName`, `className`, `fullHeight`.

Props kế thừa từ Table: `dataSource`, `dataKey`, `columns`, `selectable`, `selectedItems`, `singleSelect`, `pagination`, `pinnedColumns`, `showingColumns`, `sortBy`, `sortDir`, `border`, `loading`, `fullHeight`, `virtualized`, `canReorderRow`, `columnResizable`, `renderGroup`, `renderSummaryGroup`, `paramsForRenderCell`, `emptyRows`, `renderNoData`, và tất cả callbacks của Table.

---

### RULE-LIST-VIEW-TABLE-01: Luôn truyền `dataKey` và `columns` vì đây là props bắt buộc kế thừa từ Table

`ListViewTable` kế thừa từ `TableProps`, do đó `dataKey` và `columns` là bắt buộc. Thiếu các props này sẽ gây lỗi runtime hoặc bảng không hiển thị dữ liệu.

**Sai:**

```tsx
// ❌ Thiếu dataKey và columns — bắt buộc phải có
<ListViewTable
  title="Danh sách người dùng"
  dataSource={users}
/>
```

**Đúng:**

```tsx
// ✅ Truyền đầy đủ dataKey và columns
<ListViewTable
  title="Danh sách người dùng"
  dataKey="id"
  columns={columns}
  dataSource={users}
/>
```

---

### RULE-LIST-VIEW-TABLE-02: Sử dụng `className` cho container ngoài và `tableClassName` cho Table bên trong, không nhầm lẫn

`ListViewTable` tách riêng `className` (áp dụng cho div bao ngoài chứa cả header và table) và `tableClassName` (áp dụng cho component `Table` bên trong). Props `className` của `TableProps` đã bị loại bỏ qua `Omit<TableProps<T>, "className">`.

**Sai:**

```tsx
// ❌ Dùng className để style cho Table — sẽ chỉ áp dụng cho container ngoài
<ListViewTable
  title="Đơn hàng"
  dataKey="id"
  columns={columns}
  dataSource={orders}
  className="border rounded"
/>

// ❌ Không có prop className cho Table — sẽ bị lỗi TypeScript
<ListViewTable
  title="Đơn hàng"
  dataKey="id"
  columns={columns}
  dataSource={orders}
  className="border rounded" // style này áp dụng cho wrapper, không phải Table
/>
```

**Đúng:**

```tsx
// ✅ Dùng className cho container ngoài, tableClassName cho Table
<ListViewTable
  title="Đơn hàng"
  dataKey="id"
  columns={columns}
  dataSource={orders}
  className={cx("p-[1rem]")}
  tableClassName={cx("border rounded")}
/>
```

---

### RULE-LIST-VIEW-TABLE-03: Sử dụng `renderActionButtons` và `renderContextButtons` để hiển thị nút hành động, không tự render bên ngoài

`ListViewHeader` đã tích hợp layout cho action buttons (bên phải) và context buttons (bên trái, cạnh action buttons). Cả hai props đều là hàm trả về `ReactNode`. Nếu tự render bên ngoài sẽ mất bố cục responsive của header.

**Sai:**

```tsx
// ❌ Tự render nút bên ngoài ListViewTable — mất bố cục responsive
<div className={cx("flex justify-between")}>
  <h6>Danh sách</h6>
  <div>
    <Button>Thêm mới</Button>
  </div>
</div>
<ListViewTable
  dataKey="id"
  columns={columns}
  dataSource={data}
/>
```

**Đúng:**

```tsx
// ✅ Dùng renderActionButtons và renderContextButtons
<ListViewTable
  title="Danh sách"
  dataKey="id"
  columns={columns}
  dataSource={data}
  renderActionButtons={() => (
    <div className={cx("flex gap-[1rem]")}>
      <Button startIcon={<FontAwesomeIcon icon={faPlus} />}>Thêm mới</Button>
    </div>
  )}
  renderContextButtons={() => (
    <IconButton variant="text" size="small">
      <FontAwesomeIcon icon={faFilter} />
    </IconButton>
  )}
/>
```

---

### RULE-LIST-VIEW-TABLE-04: Sử dụng `renderHeadingOptions` để render phần tùy chọn giữa header và bảng

`renderHeadingOptions` là hàm trả về `ReactNode`, được render ngay dưới header và trên bảng dữ liệu. Khi không truyền `renderHeadingOptions`, component tự động thêm `margin-top: 0.75rem` cho bảng. Khi có `renderHeadingOptions`, margin này bị bỏ để nội dung liền mạch.

**Sai:**

```tsx
// ❌ Tự render filter bar bên ngoài ListViewTable — mất đồng bộ layout
<div className={cx("flex gap-[0.5rem]", "mb-[0.75rem]")}>
  <Input placeholder="Tìm kiếm..." />
  <Select options={filterOptions} />
</div>
<ListViewTable
  title="Danh sách"
  dataKey="id"
  columns={columns}
  dataSource={data}
/>
```

**Đúng:**

```tsx
// ✅ Dùng renderHeadingOptions để render phần tùy chọn
<ListViewTable
  title="Danh sách"
  dataKey="id"
  columns={columns}
  dataSource={data}
  renderHeadingOptions={() => (
    <div className={cx("flex gap-[0.5rem]", "py-[0.5rem]")}>
      <Input placeholder="Tìm kiếm..." />
      <Select options={filterOptions} />
    </div>
  )}
/>
```

---

### RULE-LIST-VIEW-TABLE-05: Sử dụng `title` và `subTitle` cho tiêu đề, không tự tạo heading bên ngoài

`ListViewHeader` đã xử lý layout responsive cho tiêu đề (`prose-h6`, `text-typo-primary`) và phụ đề (`prose-body2`, `text-typo-secondary`). Cả hai props đều nhận `ReactNode` nên có thể truyền JSX phức tạp.

**Sai:**

```tsx
// ❌ Tự render tiêu đề bên ngoài — không đồng bộ với layout header
<h6 className={cx("text-typo-primary prose-h6")}>Danh sách khách hàng</h6>
<p className={cx("text-typo-secondary prose-body2")}>Tổng: 100 khách hàng</p>
<ListViewTable
  dataKey="id"
  columns={columns}
  dataSource={customers}
/>
```

**Đúng:**

```tsx
// ✅ Dùng title và subTitle tích hợp
<ListViewTable
  title="Danh sách khách hàng"
  subTitle="Tổng: 100 khách hàng"
  dataKey="id"
  columns={columns}
  dataSource={customers}
/>

// ✅ Truyền ReactNode phức tạp cho title
<ListViewTable
  title={
    <div className={cx("flex items-center gap-[0.5rem]")}>
      <span>Danh sách khách hàng</span>
      <Badge count={totalCount} />
    </div>
  }
  subTitle={`Cập nhật lần cuối: ${lastUpdated}`}
  dataKey="id"
  columns={columns}
  dataSource={customers}
/>
```

---

### RULE-LIST-VIEW-TABLE-06: Kiểm soát `fullHeight` đúng cách để bảng chiếm toàn bộ chiều cao container

`fullHeight` mặc định là `true`, sẽ thêm class `h-full` cho container ngoài và truyền xuống Table. Khi đặt trong container có chiều cao cố định (flex layout), bảng sẽ tự động chiếm hết không gian. Nếu không cần chiếm hết chiều cao, hãy tắt `fullHeight`.

**Sai:**

```tsx
// ❌ Tự thêm h-full qua className — dư thừa vì fullHeight đã mặc định true
<ListViewTable
  title="Danh sách"
  dataKey="id"
  columns={columns}
  dataSource={data}
  className={cx("h-full")}
/>

// ❌ Không tắt fullHeight khi dùng trong context không cần chiếm hết chiều cao
<div>
  <ListViewTable
    title="Danh sách"
    dataKey="id"
    columns={columns}
    dataSource={shortList}
  />
  <div>Nội dung bên dưới</div>
</div>
```

**Đúng:**

```tsx
// ✅ fullHeight mặc định true — bảng chiếm hết chiều cao container
<div className={cx("flex flex-col", "h-screen")}>
  <div className={cx("h-[4rem]")}>Header</div>
  <div className={cx("flex-1 h-0")}>
    <ListViewTable
      title="Danh sách"
      dataKey="id"
      columns={columns}
      dataSource={data}
    />
  </div>
</div>

// ✅ Tắt fullHeight khi không cần chiếm hết chiều cao
<ListViewTable
  title="Danh sách"
  dataKey="id"
  columns={columns}
  dataSource={shortList}
  fullHeight={false}
/>
```

---

### RULE-LIST-VIEW-TABLE-07: Kết hợp ListViewTable với TableSettings đúng cách qua render props pattern

Khi cần tính năng cài đặt bảng (ẩn/hiện cột, ghim cột, resize), sử dụng `TableSettings` bao ngoài `ListViewTable` với render props pattern. Các callback `onChangeSelect`, `onChangeSelectAll`, `onResizeColumn`, `onResizeWidth` phải được lấy từ render props của `TableSettings`.

**Sai:**

```tsx
// ❌ Tự quản lý onChangeSelect và onResizeColumn — không đồng bộ với TableSettings
<TableSettings settings={settings} onChangeSettings={onChangeSettings}>
  {() => (
    <ListViewTable
      title="Danh sách"
      dataKey="id"
      columns={columns}
      dataSource={data}
      selectable
      selectedItems={selectedItems}
      onChangeSelect={(checked, item) => {
        // tự xử lý — không đồng bộ
      }}
    />
  )}
</TableSettings>
```

**Đúng:**

```tsx
// ✅ Lấy callbacks từ render props của TableSettings
<TableSettings
  dataKey="id"
  settings={{
    columns,
    showingColumns,
    pinnedColumns,
  }}
  onChangeSettings={{
    columns: setColumns,
    selectedItems: setSelectedItems,
    pinnedColumns: setPinnedColumns,
    showingColumns: setShowingColumns,
  }}
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
        <Button onClick={showSettings}>Cài đặt</Button>
      )}
    />
  )}
</TableSettings>
```

---

### RULE-LIST-VIEW-TABLE-08: Không truyền các Table props vào sai vị trí — tất cả Table props truyền trực tiếp vào ListViewTable

`ListViewTable` sử dụng spread operator `{...tableProps}` để chuyển tiếp tất cả props còn lại xuống `Table`. Do đó, các props như `pagination`, `selectable`, `sortBy`, `sortDir`, `loading`... phải truyền trực tiếp vào `ListViewTable`, không cần tạo Table riêng bên trong.

**Sai:**

```tsx
// ❌ Sử dụng Table riêng bên trong ListViewTable — dư thừa
<ListViewTable title="Danh sách" dataKey="id" columns={columns} dataSource={data}>
  <Table
    dataKey="id"
    columns={columns}
    dataSource={data}
    pagination={{ total: 100, currentPage: 1 }}
  />
</ListViewTable>
```

**Đúng:**

```tsx
// ✅ Truyền tất cả Table props trực tiếp vào ListViewTable
<ListViewTable
  title="Danh sách"
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
    total: 100,
    currentPage: 1,
    onNextPage: handleNextPage,
    onPrevPage: handlePrevPage,
    onChangePerPage: handleChangePerPage,
  }}
/>
```

---

## Tham chiếu cấu hình mặc định

| Prop | Giá trị mặc định | Mô tả |
|------|-------------------|-------|
| `title` | `""` | Tiêu đề hiển thị ở header |
| `subTitle` | `""` | Phụ đề hiển thị dưới tiêu đề |
| `renderActionButtons` | `() => null` | Hàm render nút hành động (bên phải header) |
| `renderContextButtons` | `() => null` | Hàm render nút ngữ cảnh (bên trái action buttons) |
| `renderHeadingOptions` | `undefined` | Hàm render phần tùy chọn giữa header và bảng |
| `fullHeight` | `true` | Container chiếm toàn bộ chiều cao (class `h-full`) |
| `className` | `undefined` | Class cho container ngoài (div bao gồm header + table) |
| `tableClassName` | `undefined` | Class cho component Table bên trong |

## Tham chiếu ListViewHeaderProps

| Prop | Kiểu | Mô tả |
|------|------|-------|
| `title` | `React.ReactNode` | Tiêu đề hiển thị với style `prose-h6 text-typo-primary` |
| `subTitle` | `React.ReactNode` | Phụ đề hiển thị với style `prose-body2 text-typo-secondary` |
| `renderContextButtons` | `() => React.ReactNode` | Render nút ngữ cảnh ở bên phải header |
| `renderActionButtons` | `() => React.ReactNode` | Render nút hành động ở bên phải header |
| `renderHeadingOptions` | `() => React.ReactNode` | Render phần tùy chọn nằm giữa header và bảng |
