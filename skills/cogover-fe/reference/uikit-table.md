## Cách sử dụng Table Component của ui-kit

Path: `ui-kit/src/components/Table`
Hook: `ui-kit/src/components/Table/hooks/useTableV2`

### Props chính

| Prop | Bắt buộc | Mô tả |
|---|---|---|
| `dataSource` | ✓ | `T[]` hoặc `Record<string\|number, T[]>` (grouped) |
| `dataKey` | ✓ | Path đến unique identifier (hỗ trợ nested `"user.id"`) |
| `columns` | ✓ | `ColumnType<T>[]` |
| `selectable` | — | Hiển thị cột checkbox/radio |
| `singleSelect` | — | Radio thay vì Checkbox |
| `selectedItems` | — | Items đang được chọn |
| `disabledSelect` | — | `(record) => boolean` |
| `disabledSelectTooltip` | — | `(record) => string` |
| `pinnedColumns` | — | `{ left?: string[], right?: string[] }` |
| `showingColumns` | — | `string[]` — keys của cột hiển thị |
| `sortBy` | — | Key cột đang sort |
| `sortDir` | — | `"asc"` \| `"desc"` \| `null` |
| `pagination` | — | `{ total, currentPage, perPage, onNextPage, onPrevPage, onChangePerPage, alwaysShow? }` |
| `virtualized` | — | `{ enabled: boolean, maxHeight: number, stickyBottomRowIndexes?: number[] }` |
| `columnResizable` | — | Cho phép resize cột |
| `canReorderRow` | — | Kéo thả hàng (không dùng với pinnedColumns) |
| `paramsForRenderCell` | — | Truyền dữ liệu bổ sung vào render functions |
| `border` | — | `{ wrapper?, horizontal?, vertical? }` — mặc định `horizontal: true` |
| `loading` | — | Skeleton 5 hàng |
| `renderNoData` | — | Custom empty state |

### ColumnType

| Prop | Mô tả |
|---|---|
| `key` | Unique, đồng thời là path lấy giá trị |
| `label` | Header text/node |
| `render` | `(record, index, dataSource, params) => ReactNode` |
| `width` | `number` (pixel) |
| `minWidth` | `number` (pixel) |
| `sortable` | `boolean` |
| `autoFillWidth` | `boolean` — co giãn fill space |
| `paddingX` | `number` — mặc định 12px |
| `canHide` | `boolean` — dùng với TableSettings |
| `canChangePosition` | `boolean` — dùng với TableSettings |

### Basic usage với useTableV2

```tsx
const {
  columns, setColumns, pinnedColumns, setPinnedColumns,
  showingColumns, setShowingColumns, page, setPage, limit, setLimit,
  sortBy, setSortBy, sortDir, setSortDir,
} = useTableV2({
  tableName: "user-list",
  initColumns: userColumns,
  initPinnedColumns: { left: ["name"], right: ["action"] },
});

<Table
  dataSource={data}
  dataKey="id"
  columns={columns}
  showingColumns={showingColumns}
  pinnedColumns={pinnedColumns}
  sortBy={sortBy}
  sortDir={sortDir}
  onSortChange={(sortBy, sortDir) => { setSortBy(sortBy); setSortDir(sortDir); }}
  selectable
  selectedItems={selectedItems}
  onChangeSelect={(checked, item) => { /* xử lý */ }}
  onChangeSelectAll={(checked, items) => { /* xử lý */ }}
  columnResizable
  onResizeColumn={(key, width) => setColumns((prev) => prev.map((c) => c.key === key ? { ...c, width } : c))}
  pagination={{ total: 100, currentPage: page, perPage: limit,
    onNextPage: () => setPage((p) => p + 1), onPrevPage: () => setPage((p) => p - 1),
    onChangePerPage: (perPage) => { setLimit(perPage); setPage(1); } }}
/>
```

### DO / DON'T

DO: Dùng `showingColumns` để ẩn/hiện cột — không filter mảng `columns`.

DO: Dùng `disabledSelect` để disable chọn có điều kiện — không lọc dataSource.

DO: Dùng `paramsForRenderCell` để truyền permissions/config vào render — tránh closure gây re-render.

DON'T: Dùng `canReorderRow` cùng `pinnedColumns` — throw Error tại runtime.

DON'T: Tự render Pagination bên ngoài — dùng prop `pagination` tích hợp.

DON'T: Tự tạo 2 Table riêng cho cột ghim — dùng `pinnedColumns`.

DON'T: `width` là number (pixel), không phải string `"200px"`.
