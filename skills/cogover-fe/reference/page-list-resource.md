## Quy tắc tạo trang danh sách (List Page) cho một resource bất kỳ

Pattern trang danh sách với bảng, sidebar filter, phân trang, sắp xếp, CRUD.

### Cấu trúc thư mục

```
src/pages/{Resource}Page/{Resource}ListPage/
├── index.tsx              // FormProvider + Layout
├── type.ts                // Enum FilterType, interface FilterForm
├── options.tsx            // Options cho filter selects
└── components/
    ├── {Resource}List/
    │   ├── index.tsx       // TableSettings + ListViewTable + CRUD
    │   ├── helper.ts       // buildParamsFromFilter()
    │   └── useGetColumns.tsx
    ├── FilterMenu/index.tsx
    └── ActionMenu/index.tsx
```

### type.ts

```ts
import { Dayjs } from 'dayjs';
export enum {Resource}FilterType { Status, Category, CreatedAt }
export interface {Resource}FilterForm {
    filterTypes: {Resource}FilterType[];
    searchValue: string;
    statuses: number | null;
    categories: string[];
    categoryOperator: string | null;
    createdAtRange: [Dayjs | null, Dayjs | null];
}
```

### index.tsx — FormProvider + URL sync

```tsx
export const ADVANCE_FILTER_STORAGE_KEY = 'advanceFilter:{resource}';
export default function {Resource}ListPage() {
    const { state: defaultValues } = useGetStateFromSearchParams<{Resource}FilterForm>({
        getState: (searchParams) => ({
            filterTypes: JSON.parse(localStorage.getItem(ADVANCE_FILTER_STORAGE_KEY) ?? '[]'),
            searchValue: searchParams.searchValue ?? '',
        }),
    });
    const methods = useForm<{Resource}FilterForm>({ values: defaultValues });
    const [searchValue, statuses] = useWatch({ control: methods.control, name: ['searchValue', 'statuses'] });
    const savedDataToUrl = useMemo(() => ({ searchValue: searchValue.trim(), statuses }), [searchValue, statuses]);
    return (
        <FormProvider {...methods}>
            <SyncFilterFormWithSearchParams data={savedDataToUrl}>
                <div className={cx('flex px')}>
                    <div className={cx('flex-1 flex flex-col gap-[1rem] h-[calc(100svh-var(--top-toolbar-width))] w-0 px-[1.5rem] py-[1rem]')}>
                        <div className={cx('flex-1 h-0')}><{Resource}List /></div>
                    </div>
                    <ResizableMenu storageKey='{resource}' saveToLocalStorage collapsedIcon={({ expandMenu }) => (
                        <button className={cx('flex justify-center mt-[1rem] w-full')} onClick={expandMenu}><FontAwesomeIcon icon={faFilterList} fontSize={20} /></button>
                    )}>
                        <FilterMenu />
                    </ResizableMenu>
                </div>
            </SyncFilterFormWithSearchParams>
        </FormProvider>
    );
}
```

### {Resource}List/index.tsx — TableSettings + ListViewTable

```tsx
export default function {Resource}List() {
    const [showConfirmDelete, setShowConfirmDelete] = useState(false);
    const deletedItems = useRef<{Resource}[]>([]);
    const onClickDelete = useCallback((item: {Resource}) => { deletedItems.current = [item]; setShowConfirmDelete(true); }, []);
    const { columns: initColumns } = useGetColumns({ onClickDelete });
    const { columns, setColumns, pinnedColumns, setPinnedColumns, showingColumns, setShowingColumns, sortBy, setSortBy, sortDir, setSortDir, page, setPage, limit, setLimit } =
        useTableV2({ tableName: '{resource}', initColumns, initPinnedColumns: { left: [], right: ['action'] } });
    const formValues = useWatch({ control: useFormContext<{Resource}FilterForm>().control }) as {Resource}FilterForm;
    const [debouncedParams] = useDebounceValue(formValues, 800);
    useEffect(() => { setPage(1); }, [debouncedParams, setPage]);
    const params = useMemo(() => buildParamsFromFilter({ ...debouncedParams, page, limit, sortBy, sortDir }), [debouncedParams, page, limit, sortBy, sortDir]);
    const { data: response, isPending } = useQuery{Resource}List(params);
    const { data = [], meta } = response ?? {};
    const { mutate: deleteItem, isPending: isDeleting } = useDelete{Resource}({
        onSuccess: () => { setShowConfirmDelete(false); showToastMessage({ type: 'success', title: '...', message: '...' }); },
    });
    return (
        <>
            <TableSettings dataKey="id" tableName="{resource}" settings={{ columns, pinnedColumns, showingColumns }}
                onChangeSettings={{ columns: setColumns, pinnedColumns: setPinnedColumns, showingColumns: setShowingColumns }}
            >
                {({ onResizeColumn, showSettings, onChangeSelect, onChangeSelectAll, onResizeWidth }) => (
                    <ListViewTable title={`Danh sách (${meta?.total ?? 0})`} stickyHeader columnResizable
                        columns={columns} pinnedColumns={pinnedColumns} showingColumns={showingColumns}
                        dataKey="id" dataSource={data} sortBy={sortBy} sortDir={sortDir} loading={isPending}
                        onResizeColumn={onResizeColumn} onChangeSelect={onChangeSelect} onChangeSelectAll={onChangeSelectAll} onResizeWidth={onResizeWidth}
                        onSortChange={(sortBy, sortDir) => { setSortBy(sortBy); setSortDir(sortDir); }}
                        renderActionButtons={() => <Button startIcon={<FontAwesomeIcon icon={faSliders} />} variant="gray" onClick={showSettings}>Cài đặt</Button>}
                        renderNoData={() => <h5 className={cx('prose-h5 text-typo-secondary mt-[2.5rem]')}>Không có dữ liệu</h5>}
                        pagination={{ currentPage: page, total: meta?.total ?? 0, perPage: limit,
                            onNextPage: () => setPage((p) => p + 1), onPrevPage: () => setPage((p) => p - 1),
                            onChangePerPage: (limit) => { setPage(1); setLimit(limit); }, perPageOptions: TABLE_PAGE_SIZE_OPTIONS }}
                    />
                )}
            </TableSettings>
            {showConfirmDelete && (
                <ConfirmModal title="Xóa bản ghi" loading={isDeleting}
                    message={<span>Bạn có chắc chắn muốn xóa <strong>{deletedItems.current[0].name}</strong>?</span>}
                    onClose={() => setShowConfirmDelete(false)}
                    onConfirm={() => deleteItem({ ids: deletedItems.current.map((item) => item.id) })}
                />
            )}
        </>
    );
}
```

### helper.ts — buildParamsFromFilter

```ts
export const buildParamsFromFilter = (filter: {Resource}FilterForm & { limit: number; page: number; sortBy: string | null; sortDir: string | null }): GetListParams => {
    const params: GetListParams = { page: filter.page, limit: filter.limit };
    if (filter.searchValue.trim()) { params.name = filter.searchValue.trim(); params.nameOperator = 'LIKE'; }
    if (filter.filterTypes.includes(FilterType.Category) && filter.categories.length) {
        params.categoryId = filter.categories.join(',');
        params.categoryIdOperator = filter.categoryOperator === 'notEqual' ? 'NOT_IN' : 'IN';
    }
    if (filter.filterTypes.includes(FilterType.CreatedAt) && filter.createdAtRange.some(Boolean)) {
        filter.createdAtRange[0] && (params.createdStart = filter.createdAtRange[0].set('hour', 0).set('minute', 0).set('second', 0).valueOf());
        filter.createdAtRange[1] && (params.createdEnd = filter.createdAtRange[1].set('hour', 23).set('minute', 59).set('second', 59).valueOf());
    }
    if (filter.sortBy) { params.order = filter.sortBy; params.sort = filter.sortDir ?? 'asc'; }
    return params;
};
```

### Quy tắc quan trọng

- `useRef` lưu item đang thao tác, `useState` chỉ cho visibility modal
- `useEffect` reset page = 1 khi debounced params thay đổi (800ms debounce)
- Guard params: `filterTypes.includes()` + `&& value.length` — không gửi param rỗng
- Date range: normalize (start = 00:00:00, end = 23:59:59)
- FilterMenu: `appendToBody={false}` cho select trong sidebar; onChange → `syncFilterFormWithSearchParams()`
- ActionMenu: spread `{...params} {...attributes}` (params trước attributes)
