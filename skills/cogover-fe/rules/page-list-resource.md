---
title: Quy tắc tạo trang danh sách (List Page) cho một resource bất kỳ
impact: HIGH
impactDescription: Trang danh sách là pattern phổ biến nhất trong dự án. Nếu không tuân theo quy tắc, code sẽ không nhất quán, khó bảo trì, thiếu tính năng URL sync và filter persistence
tags: page, list, table, filter, crud, resource, pattern, architecture
---

## Quy tắc tạo trang danh sách (List Page) cho một resource bất kỳ

Hướng dẫn tạo trang danh sách với bảng dữ liệu, bộ lọc sidebar, phân trang, sắp xếp, và các hành động CRUD.

> **Quan trọng:** Chỉ cần làm theo hướng dẫn trong rule này. Không cần tham khảo các trang list đã có trong dự án trừ khi user yêu cầu rõ ràng.

---

## Thông tin cần có từ user

Khi được yêu cầu tạo page dạng list, **phải hỏi lại user** các thông tin sau trước khi bắt đầu:

1. **Tên resource** — tên resource/entity (ví dụ: `Campaign`, `Contact`, `Template`)
2. **Thông tin API** — request API (URL, method, params) và response API (cấu trúc response, các trường của bản ghi) → dùng để tạo hook TanStack Query
3. **Các trường cần filter** — danh sách filter fields và kiểu filter (select, multi-select, date range, text search...)
4. **Các action của từng record** — danh sách actions trên mỗi row (xem chi tiết, sửa, xóa, nhân bản...)

**Tùy chọn (dùng mặc định nếu user không cung cấp):**
- Permission (`functionCode`, `targetAction`) → bỏ qua, thêm sau
- Sort mặc định → `created` / `desc`
- Route path → dựa theo tên resource
- i18n → viết tiếng Việt trước, dịch sau

---

## RULE-LIST-PAGE-01: Cấu trúc thư mục

Mỗi List Page phải tuân theo cấu trúc thư mục sau:

**Sai:**

```
// ❌ Gộp tất cả vào 1 file hoặc đặt flat không phân nhóm
src/pages/UserPage/
├── UserListPage.tsx        // Mọi thứ trong 1 file
├── columns.ts
└── types.ts
```

**Đúng:**

```
// ✅ Tách rõ concern theo thư mục
src/pages/{Resource}Page/
├── {Resource}ListPage/
│   ├── index.tsx              // Trang chính: FormProvider + Layout
│   ├── type.ts                // Enum FilterType, interface FilterForm
│   ├── options.tsx            // Mảng options cho filter (select, status...)
│   └── components/
│       ├── {Resource}List/
│       │   ├── index.tsx       // Bảng dữ liệu: TableSettings + ListViewTable + CRUD
│       │   ├── helper.ts       // buildParamsFromFilter() — chuyển form → API params
│       │   └── useGetColumns.tsx  // Hook định nghĩa cột bảng
│       ├── FilterMenu/
│       │   └── index.tsx       // Sidebar bộ lọc
│       └── ActionMenu/
│           └── index.tsx       // Menu hành động cho từng row (Popper)
```

---

## RULE-LIST-PAGE-02: Tầng Type (`type.ts`)

Phải định nghĩa enum cho filter types và interface cho filter form. Mỗi filter field có operator đi kèm nếu cần `IN`/`NOT_IN`.

**Sai:**

```ts
// ❌ Không có type, dùng inline object
const defaultValues = {
  search: '',
  status: null,
  categories: [],
};
```

**Đúng:**

```ts
// ✅ type.ts
import { Dayjs } from 'dayjs';

export enum {Resource}FilterType {
    Status,
    Category,
    CreatedBy,
    CreatedAt,
    // ... thêm filter khác
}

export interface {Resource}FilterForm {
    filterTypes: {Resource}FilterType[];   // filter nào đang bật
    searchValue: string;                    // ô tìm kiếm
    statuses: number | null;               // select đơn
    categories: string[];                  // select nhiều
    categoryOperator: string | null;       // toán tử IN/NOT_IN
    createdBys: string[];
    createdByOperator: string | null;
    createdAtRange: [Dayjs | null, Dayjs | null]; // khoảng ngày
    // ...
}
```

**Quy tắc:**
- `filterTypes` array quyết định filter nào đang active trên UI
- Date range luôn dùng tuple `[Dayjs | null, Dayjs | null]`
- Mỗi multi-select field có 1 field operator đi kèm (`{field}Operator`)

---

## RULE-LIST-PAGE-03: Options (`options.tsx`)

Tách riêng file options, label luôn là i18n key.

**Sai:**

```ts
// ❌ Hardcode text tiếng Việt, định nghĩa inline trong component
const options = [
  { label: 'Trạng thái', value: 0 },
  { label: 'Danh mục', value: 1 },
];
```

**Đúng:**

```tsx
// ✅ options.tsx — label là i18n key, tách riêng file
import { {Resource}FilterType, {Resource}Status } from './type';

export const filterOptions: { label: string; value: {Resource}FilterType }[] = [
    { label: 'ns:key.status', value: {Resource}FilterType.Status },
    { label: 'ns:key.category', value: {Resource}FilterType.Category },
    // ...
];

export const statusOptions: { label: string; value: {Resource}Status }[] = [
    { label: 'ns:key.active', value: {Resource}Status.Active },
    { label: 'ns:key.inactive', value: {Resource}Status.Inactive },
];
```

---

## RULE-LIST-PAGE-04: Trang chính (`index.tsx`) — FormProvider + URL Sync + Layout

Trang chính phải bọc trong `FormProvider` và đồng bộ filter state với URL.

**Sai:**

```tsx
// ❌ Không có FormProvider, không sync URL, state nằm rải rác
export default function ResourceListPage() {
    const [search, setSearch] = useState('');
    const [status, setStatus] = useState(null);

    return (
        <div>
            <input value={search} onChange={(e) => setSearch(e.target.value)} />
            <ResourceTable search={search} status={status} />
        </div>
    );
}
```

**Đúng:**

```tsx
// ✅ index.tsx
import { useGetStateFromSearchParams } from '@stringeecom/ui-kit';
import { FormProvider, useForm, useWatch } from 'react-hook-form';
import { ResizableMenu } from 'src/components/ResizableMenu';
import SyncFilterFormWithSearchParams from 'src/components/SyncFilterFormWithSearchParams';
import cx from 'src/utils/cx';

export const ADVANCE_FILTER_STORAGE_KEY = 'advanceFilter:{resource}';

export default function {Resource}ListPage() {
    const { t } = useTranslation(I18nNS.{RESOURCE});
    useDocumentTitle(`${t('{resource}List.title')} - Cogover`);

    // 1. Đọc URL params → tạo defaultValues
    const { state: defaultValues } = useGetStateFromSearchParams<{Resource}FilterForm>({
        getState: (searchParams) => {
            const persistedFilter = window.localStorage.getItem(ADVANCE_FILTER_STORAGE_KEY);
            return {
                filterTypes: persistedFilter
                    ? (JSON.parse(persistedFilter) as {Resource}FilterType[])
                    : [/* default filter types */],
                searchValue: searchParams.searchValue ?? '',
                // ... parse các field khác từ searchParams
            };
        },
    });

    // 2. Tạo form với react-hook-form
    const methods = useForm<{Resource}FilterForm>({ values: defaultValues });

    // 3. Watch form values để sync lên URL
    const [searchValue, statuses, /* ... */] = useWatch({
        control: methods.control,
        name: ['searchValue', 'statuses', /* ... */],
    });

    // 4. Tính savedDataToUrl bằng useMemo
    const savedDataToUrl = useMemo(() => ({
        searchValue: searchValue.trim(),
        statuses,
        // ... chỉ gửi param của filter đang bật
    }), [searchValue, statuses, /* ... */]);

    // 5. Render layout
    return (
        <FormProvider {...methods}>
            <SyncFilterFormWithSearchParams data={savedDataToUrl}>
                <div className={cx('flex', 'px')}>
                    <div className={cx(
                        'flex-1 flex flex-col gap-[1rem]',
                        'h-[calc(100svh-var(--top-toolbar-width))]',
                        'w-0 px-[1.5rem] py-[1rem]',
                    )}>
                        <div className={cx('flex-1 h-0')}>
                            <{Resource}List />
                        </div>
                    </div>
                    <ResizableMenu
                        collapsedIcon={({ expandMenu }) => (
                            <button className={cx('flex justify-center', 'mt-[1rem] w-full')} onClick={expandMenu}>
                                <FontAwesomeIcon icon={faFilterList} fontSize={20} />
                            </button>
                        )}
                        storageKey='{resource}'
                        saveToLocalStorage
                    >
                        <FilterMenu />
                    </ResizableMenu>
                </div>
            </SyncFilterFormWithSearchParams>
        </FormProvider>
    );
}
```

**Cấu trúc layout:**
```
FormProvider
  └── SyncFilterFormWithSearchParams (đồng bộ form ↔ URL)
        └── flex container
              ├── Bảng dữ liệu (flex-1 w-0)
              └── ResizableMenu (sidebar filter, co giãn)
```

---

## RULE-LIST-PAGE-05: Bảng dữ liệu (`{Resource}List/index.tsx`)

Bảng phải dùng `TableSettings` + `ListViewTable`, debounce filter, và quản lý state bằng `useTableV2`.

**Sai:**

```tsx
// ❌ Không debounce, tự quản lý state, không dùng TableSettings
export default function ResourceList() {
    const [page, setPage] = useState(1);
    const { data } = useQuery(['list', formValues], () => api.getList(formValues));

    return <Table dataSource={data} columns={columns} />;
}
```

**Đúng:**

```tsx
// ✅ {Resource}List/index.tsx
export default function {Resource}List() {
    // --- State cho modal ---
    const [showConfirmDelete, setShowConfirmDelete] = useState(false);
    const deletedItems = useRef<{Resource}[]>([]);

    // --- Callbacks cho ActionMenu ---
    const onClickDelete = useCallback((item: {Resource}) => {
        deletedItems.current = [item];
        setShowConfirmDelete(true);
    }, []);

    // --- Columns ---
    const { columns: initColumns } = useGetColumns({ onClickDelete });

    // --- Table state (useTableV2) ---
    const {
        columns, setColumns,
        pinnedColumns, setPinnedColumns,
        showingColumns, setShowingColumns,
        sortBy, setSortBy, sortDir, setSortDir,
        page, setPage, limit, setLimit,
    } = useTableV2({
        tableName: '{resource}',
        initColumns,
        initPinnedColumns: { left: [], right: ['action'] },
    });

    // --- Form values + Debounce ---
    const { control } = useFormContext<{Resource}FilterForm>();
    const formValues = useWatch({ control }) as {Resource}FilterForm;
    const [debouncedParams] = useDebounceValue(formValues, 800);

    // Reset page khi filter thay đổi
    useEffect(() => { setPage(1); }, [debouncedParams, setPage]);

    // --- Build API params ---
    const params = useMemo(
        () => buildParamsFromFilter({ ...debouncedParams, page, limit, sortBy, sortDir }),
        [debouncedParams, page, limit, sortBy, sortDir],
    );

    // --- Fetch data ---
    const { data: response, isPending } = useQuery{Resource}List(params);
    const { data = [], meta } = response ?? {};

    // --- Mutations ---
    const { mutate: deleteItem, isPending: isDeleting } = useDelete{Resource}({
        onSuccess: () => {
            setShowConfirmDelete(false);
            showToastMessage({ type: 'success', title: '...', message: '...' });
        },
    });

    // --- Render ---
    return (
        <>
            <TableSettings
                dataKey="id"
                tableName="{resource}"
                settings={{ columns, pinnedColumns, showingColumns }}
                onChangeSettings={{
                    columns: setColumns,
                    pinnedColumns: setPinnedColumns,
                    showingColumns: setShowingColumns,
                }}
            >
                {({ onResizeColumn, showSettings, onChangeSelect, onChangeSelectAll, onResizeWidth }) => (
                    <ListViewTable
                        title={`Danh sách (${meta?.total ?? 0})`}
                        stickyHeader
                        columnResizable
                        columns={columns}
                        pinnedColumns={pinnedColumns}
                        showingColumns={showingColumns}
                        dataKey="id"
                        dataSource={data}
                        sortBy={sortBy}
                        sortDir={sortDir}
                        loading={isPending}
                        onResizeColumn={onResizeColumn}
                        onChangeSelect={onChangeSelect}
                        onChangeSelectAll={onChangeSelectAll}
                        onResizeWidth={onResizeWidth}
                        onSortChange={(sortBy, sortDir) => {
                            setSortBy(sortBy);
                            setSortDir(sortDir);
                        }}
                        renderActionButtons={() => (
                            <div className={cx('flex items-center gap-[0.5rem]')}>
                                <Button startIcon={<FontAwesomeIcon icon={faSliders} />} variant="gray" onClick={showSettings}>
                                    Cài đặt
                                </Button>
                                {/* Nút tạo mới, import... */}
                            </div>
                        )}
                        renderNoData={() => (
                            <h5 className={cx('prose-h5 text-typo-secondary', 'mt-[2.5rem]')}>
                                Không có dữ liệu
                            </h5>
                        )}
                        pagination={{
                            currentPage: page,
                            total: meta?.total ?? 0,
                            perPage: limit,
                            onNextPage: () => setPage((prev) => prev + 1),
                            onPrevPage: () => setPage((prev) => prev - 1),
                            onChangePerPage: (limit) => { setPage(1); setLimit(limit); },
                            perPageOptions: TABLE_PAGE_SIZE_OPTIONS,
                        }}
                    />
                )}
            </TableSettings>

            {/* Confirm delete */}
            {showConfirmDelete && (
                <ConfirmModal
                    title="Xóa bản ghi"
                    message={<span>Bạn có chắc chắn muốn xóa <strong>{deletedItems.current[0].name}</strong>?</span>}
                    loading={isDeleting}
                    onClose={() => setShowConfirmDelete(false)}
                    onConfirm={() => deleteItem({ ids: deletedItems.current.map((item) => item.id) })}
                />
            )}
        </>
    );
}
```

**Flow dữ liệu:**
```
formValues (useWatch)
  → useDebounceValue (800ms)
    → buildParamsFromFilter (useMemo)
      → useQuery{Resource}List (React Query)
        → ListViewTable (render)
```

**Quy tắc:**
- `useRef` lưu item đang thao tác (delete/duplicate), `useState` chỉ cho visibility modal
- `useEffect` reset page = 1 khi debounced params thay đổi (đây là side effect hợp lệ)
- `useSaveToSearchParams` để sync sort lên URL

---

## RULE-LIST-PAGE-06: Helper chuyển form → API params (`helper.ts`)

Hàm `buildParamsFromFilter` phải guard bằng `filterTypes.includes()` và chỉ gửi param có giá trị.

**Sai:**

```ts
// ❌ Gửi tất cả params kể cả rỗng, không guard filterTypes
export const buildParams = (filter: FilterForm) => ({
    name: filter.searchValue,
    status: filter.statuses,
    categoryId: filter.categories.join(','),
    createdStart: filter.createdAtRange[0]?.valueOf(),
});
```

**Đúng:**

```ts
// ✅ helper.ts
export const buildParamsFromFilter = (
    filter: {Resource}FilterForm & { limit: number; page: number; sortBy: string | null; sortDir: string | null },
): GetListParams => {
    const params: GetListParams = { page: filter.page, limit: filter.limit };

    // Tìm kiếm — chỉ gửi khi có giá trị
    if (filter.searchValue.trim()) {
        params.name = filter.searchValue.trim();
        params.nameOperator = 'LIKE';
    }

    // Select — guard bằng filterTypes.includes() VÀ có giá trị
    if (filter.filterTypes.includes(FilterType.Category) && filter.categories.length) {
        params.categoryId = filter.categories.join(',');
        params.categoryIdOperator = filter.categoryOperator === 'notEqual' ? 'NOT_IN' : 'IN';
    }

    // Date range — normalize giờ: start = 00:00:00, end = 23:59:59
    if (filter.filterTypes.includes(FilterType.CreatedAt) && filter.createdAtRange.some(Boolean)) {
        filter.createdAtRange[0] && (params.createdStart = filter.createdAtRange[0]
            .set('hour', 0).set('minute', 0).set('second', 0).set('millisecond', 0).valueOf());
        filter.createdAtRange[1] && (params.createdEnd = filter.createdAtRange[1]
            .set('hour', 23).set('minute', 59).set('second', 59).set('millisecond', 0).valueOf());
    }

    // Sort
    if (filter.sortBy) {
        params.order = filter.sortBy;
        params.sort = filter.sortDir ?? 'asc';
    }

    return params;
};
```

**Quy tắc:**
- Guard bằng `filterTypes.includes()` — chỉ gửi param của filter đang bật
- Guard thêm `&& value` / `&& array.length` — không gửi param rỗng
- Date range: normalize giờ (start = 00:00:00, end = 23:59:59)
- Trả về object sạch, không có key `undefined`

---

## RULE-LIST-PAGE-07: Định nghĩa cột (`useGetColumns.tsx`)

Cột bảng phải được định nghĩa trong custom hook, wrap trong `useMemo`.

**Sai:**

```tsx
// ❌ Định nghĩa columns trực tiếp trong component, không có useMemo
function ResourceList() {
    const columns = [
        { key: 'name', label: 'Tên' },
        { key: 'action', label: '', render: (record) => <button onClick={() => delete(record)}>Xóa</button> },
    ];
    return <Table columns={columns} />;
}
```

**Đúng:**

```tsx
// ✅ useGetColumns.tsx
interface Props {
    onClickDuplicate: (item: {Resource}) => void;
    onClickDelete: (item: {Resource}) => void;
}

export default function useGetColumns(props: Props) {
    const { onClickDuplicate, onClickDelete } = props;
    const { t } = useTranslation(I18nNS.{RESOURCE});

    const columns = useMemo<ColumnType<{Resource}>[]>(() => [
        {
            key: 'name',
            label: t('{resource}List.name'),
            width: 400,
            sortable: true,
            autoFillWidth: true,
            render: (record) => (
                <SmartTooltip content={record.name} delay={[500, 0]}>
                    {({ getRef }) => (
                        <TextLink component={Link} to={`/path/${record.id}`} ref={getRef}
                            className={cx('inline-block w-full', 'whitespace-nowrap text-ellipsis overflow-hidden')}>
                            {record.name}
                        </TextLink>
                    )}
                </SmartTooltip>
            ),
        },
        {
            key: 'createdBy',
            label: t('{resource}List.creator'),
            width: 235,
            sortable: true,
            render: (record) => (
                <CreatedByInfo avatar={record.createdBy?.avatar} id={record.createdBy?.id} name={record.createdBy?.fullName} resolutionSize="small" />
            ),
        },
        {
            key: 'created',
            label: t('{resource}List.createdAt'),
            width: 180,
            sortable: true,
            render: (record) => <DateTimeWithTooltip time={record.created} />,
        },
        {
            key: 'action',
            settingLabel: t('common:common.action'),
            width: 40,
            minWidth: 40,
            paddingX: 0,
            render: (record) => (
                <ActionMenu
                    record={record}
                    onClickDuplicate={() => onClickDuplicate(record)}
                    onClickDelete={() => onClickDelete(record)}
                />
            ),
        },
    ], [onClickDelete, onClickDuplicate, t]);

    return { columns };
}
```

**Quy tắc:**
- Là custom hook, nhận action callbacks qua props
- `columns` wrap trong `useMemo` vì là array reference
- Cột `action`: pinned right, `width: 40`, `minWidth: 40`, `paddingX: 0`
- Cột name: nên có `autoFillWidth: true`
- Dùng `SmartTooltip` cho text có thể bị cắt
- Dùng `CreatedByInfo` cho cột người tạo/cập nhật
- Dùng `DateTimeWithTooltip` cho cột thời gian

---

## RULE-LIST-PAGE-08: Sidebar Filter (`FilterMenu/index.tsx`)

Filter fields phải render có điều kiện theo `filterTypes` và sync thay đổi lên URL.

**Sai:**

```tsx
// ❌ Hiển thị tất cả filter cùng lúc, không sync URL
function FilterMenu() {
    const [search, setSearch] = useState('');
    const [status, setStatus] = useState(null);
    return (
        <div>
            <input value={search} onChange={(e) => setSearch(e.target.value)} />
            <select value={status} onChange={(e) => setStatus(e.target.value)}>...</select>
        </div>
    );
}
```

**Đúng:**

```tsx
// ✅ FilterMenu/index.tsx
export default function FilterMenu() {
    const { control, setValue, reset } = useFormContext<{Resource}FilterForm>();
    const { t } = useTranslation(I18nNS.{RESOURCE});
    const { syncFilterFormWithSearchParams, setSearchParams } = useSyncFilterFormWithSearchParams();

    const [filterTypes] = useWatch({ control, name: ['filterTypes'] });

    return (
        <div className={cx('px-[1rem] py-[0.75rem]', 'h-[calc(100svh-var(--top-toolbar-width))]', 'overflow-auto')}>
            {/* Tab header */}
            <Tabs
                tabs={[
                    <button key="filter" className={cx('flex items-center gap-[0.375rem]')}>
                        <FontAwesomeIcon icon={faFilterList} fontSize={16} />
                        <span className={cx('prose-body1', 'translate-y-[1px]')}>
                            Bộ lọc {!!filterTypes.length && `(${filterTypes.length})`}
                        </span>
                    </button>,
                ]}
                currentTabIndex={0}
            />

            {/* Chọn loại filter */}
            <div className={cx('mt-[1rem]')}>
                <ControlledMultipleSelect
                    name="filterTypes"
                    options={filterOptions}
                    getLabel={(option) => t(option.label)}
                    tagVariant="gray"
                    appendToBody={false}
                    fullWidth
                    onChange={(options) => {
                        window.localStorage.setItem(
                            ADVANCE_FILTER_STORAGE_KEY,
                            JSON.stringify(options.map((o) => o.value)),
                        );
                    }}
                />
            </div>

            {/* Nút reset */}
            <div className={cx('flex justify-end', 'mt-[1rem]')}>
                <button
                    className={cx('prose-caption2 text-typo-secondary underline')}
                    onClick={() => { setSearchParams(); reset(); }}
                >
                    Đặt lại
                </button>
            </div>

            {/* Ô tìm kiếm — luôn hiển thị */}
            <div className={cx('mt-[1rem]')}>
                <ControlledTextField
                    fullWidth
                    name="searchValue"
                    onChange={() => syncFilterFormWithSearchParams()}
                    placeholder="Tên..."
                    startAdornment={<FontAwesomeIcon icon={faSearch} />}
                />
            </div>

            {/* Filter có điều kiện theo filterTypes */}
            {filterTypes.includes({Resource}FilterType.Status) && (
                <div className={cx('mt-[1rem]')}>
                    <FormItem
                        label="Trạng thái"
                        noOutLine
                        component={
                            <ControlledSelect
                                name="statuses"
                                options={statusOptions}
                                appendToBody={false}
                                onChange={() => syncFilterFormWithSearchParams()}
                                searchable={false}
                                getLabel={(option) => t(option.label)}
                                fullWidth
                                clearable
                                showBorderOnBlur={false}
                            />
                        }
                    />
                </div>
            )}

            {/* ... thêm filter khác tương tự ... */}
        </div>
    );
}
```

**Quy tắc:**
- Dùng `useFormContext` lấy form từ `FormProvider` cha
- Filter fields render **có điều kiện** theo `filterTypes.includes()`
- Mỗi field thay đổi → gọi `syncFilterFormWithSearchParams()` để cập nhật URL
- Nút reset: `setSearchParams()` + `reset()` form
- `appendToBody={false}` cho select/popper nằm trong sidebar
- `showBorderOnBlur={false}` cho style clean

---

## RULE-LIST-PAGE-09: Action Menu (`ActionMenu/index.tsx`)

Menu hành động phải dùng `Popper` + `PopperMenu` + `PopperMenuItem`.

**Sai:**

```tsx
// ❌ Tự build dropdown menu
function ActionMenu({ record, onDelete }) {
    const [open, setOpen] = useState(false);
    return (
        <div>
            <button onClick={() => setOpen(!open)}>⋮</button>
            {open && (
                <div className="dropdown">
                    <button onClick={onDelete}>Xóa</button>
                </div>
            )}
        </div>
    );
}
```

**Đúng:**

```tsx
// ✅ ActionMenu/index.tsx
interface Props {
    record: {Resource};
    onClickDuplicate: () => void;
    onClickDelete: () => void;
}

export default function ActionMenu(props: Props) {
    const { record, onClickDuplicate, onClickDelete } = props;
    const { t } = useTranslation(I18nNS.{RESOURCE});

    return (
        <Popper
            placement="bottom-end"
            render={({ attributes, ...params }) => (
                <PopperMenu {...params} {...attributes} className="hide-on-click">
                    <PopperMenuItem component={Link} to={`/path/${record.id}`}>
                        <FontAwesomeIcon icon={faEye} />
                        Xem chi tiết
                    </PopperMenuItem>
                    <PopperMenuItem onClick={onClickDuplicate}>
                        <FontAwesomeIcon icon={faClone} />
                        Nhân bản
                    </PopperMenuItem>
                    <PopperMenuItem onClick={onClickDelete}>
                        <FontAwesomeIcon icon={faTrashCan} />
                        Xóa
                    </PopperMenuItem>
                </PopperMenu>
            )}
        >
            {(params, isOpen) => (
                <IconButton
                    variant="text"
                    className={cx('text-typo-primary', { 'border-info': isOpen })}
                    {...params}
                >
                    <FontAwesomeIcon icon={faEllipsisVertical} />
                </IconButton>
            )}
        </Popper>
    );
}
```

**Quy tắc:**
- Spread thứ tự: `{...params} {...attributes}` (params trước, theo RULE-POPPER-02)
- `className='hide-on-click'` tự đóng popper khi click item
- Khi có permission: dùng `Can` component cho action-level, biến boolean cho record-level
- Tính toán quyền trực tiếp (không cần `useMemo` cho giá trị derived đơn giản)

---

## RULE-LIST-PAGE-10: Tổng kết flow dữ liệu

```
URL params
  ↓ useGetStateFromSearchParams
Form defaultValues
  ↓ useForm + FormProvider
┌─────────────────────────────────────┐
│  FilterMenu (sidebar)               │ ← user thay đổi filter
│    → syncFilterFormWithSearchParams()│
│    → form values update              │
└─────────────────────────────────────┘
  ↓ useWatch
  ↓ useDebounceValue (800ms)
  ↓ buildParamsFromFilter (useMemo)
  ↓ useQuery{Resource}List (React Query)
┌─────────────────────────────────────┐
│  ListViewTable                       │ → hiển thị data
│    ├── Pagination                    │ → page, limit
│    ├── Sort                          │ → sortBy, sortDir → URL
│    └── ActionMenu                    │ → CRUD actions
│         ├── ConfirmModal (xóa)       │
│         └── Modal (nhân bản)         │
└─────────────────────────────────────┘
```

**Nguyên tắc chung:**
- **Single source of truth**: Form state (react-hook-form) là trung tâm
- **URL là bookmark**: Mọi filter/sort đều sync lên URL → share/refresh giữ nguyên state
- **Debounce trước API call**: Tránh spam request khi user đang nhập
- **Tách concern**: type → options → helper → columns → component
- **Ref cho modal data**: Dùng `useRef` lưu item đang thao tác, `useState` chỉ cho visibility
- **Permission-aware**: Dùng `Can` component + access control trên record (thêm sau nếu chưa có)
