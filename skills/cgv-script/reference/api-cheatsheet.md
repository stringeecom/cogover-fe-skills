# API Cheatsheet — Layout Rule Script

Knowledge base bắt buộc khi sinh code cho Layout Rule Script.

## 1. Biến global (sẵn có, không khai báo)

| Biến | Kiểu | Mô tả |
|---|---|---|
| `screen` | object | Snapshot màn hình hiện tại |
| `initScreen` | object | Snapshot lúc load lần đầu |
| `$record` | object | Field slug → value của bản ghi đang sửa. **`$record.id`** có → form đã lưu |
| `$parentRecord` | object | Bản ghi cha (khi form con) |
| `$currentPersonnel` | object | User đang đăng nhập |
| `changedFields` | object | `{ slug: true }` cho field user vừa đổi |
| `changedRowCells` | array | `[{ relatedListSlug, rowIndex, recordId, fieldSlug, newValue }, ...]`. `newValue` đã được convert sang SCRIPT shape — đồng nhất với `row.get(slug).value` / `$record.X` (vd lookup single → string ID, date → Date, time → ms). |
| `isDirtyForm` | boolean | Form có thay đổi chưa lưu |
| `locale` | string | `vi`, `en`, ... |
| `$ref` | object | Bộ nhớ tạm script — giữ qua các lần exec trong cùng instance |
| `formType` | `"view" \| "create"` | Auto compute: `"view"` nếu `$record.id` có, `"create"` nếu không. Dùng gate logic theo mode. |
| `filterRecords(slug, params)` | async fn | Query record từ object khác |

## 2. screen.get(slug, type)

```js
screen.get("name", "FORM_ITEM")      // 1 field
screen.get("items", "RELATED_LIST")  // 1 bảng
screen.get("section_1", "SECTION")   // container
```

Type khác: `ROW`, `COLUMN`, `GROUP`, `DISPLAY_BOX`, `BUTTON`, `BUTTON_GROUP`, `PATH_COMPONENT`, `TAB`.

Action:
```js
screen.changeLayout(layoutId);
screen.triggerButton(buttonSlug, requiredFlag);
```

## 3. Form Item (field)

| Property | R/W | Mô tả |
|---|---|---|
| `slug` | R | |
| `display` | R/W | true=hiện, false=ẩn |
| `readOnly` | R/W | true=khoá |
| `required` | R/W | true=bắt buộc |
| `value` | R/W | giá trị (SCRIPT shape, xem §6) |
| `limitedOptions` | R/W | mảng option cho select |

## 4. Container (row/column/section/...)

| Property | R/W |
|---|---|
| `slug`, `display` | R/`display` cũng R/W |

## 5. Related List

| API | Sync/Async | Mô tả |
|---|---|---|
| `rl.display = bool` | sync | hiện/ẩn bảng |
| `rl.readOnly = bool` | sync | khoá bảng |
| `rl.creatableNewRecord = bool` | sync | cho thêm dòng mới |
| `rl.setData(records)` | sync | thay toàn bộ dòng (id tự gen nếu không truyền) |
| `await rl.rows()` | async | snapshot all rows `[{ rowIndex, recordId, get(slug) }]` |
| `await rl.row(i)` | async | row index i hoặc null |
| `await rl.findRow(pred)` | async | tìm theo predicate |
| `await rl.submit()` | async | tự lưu bảng (batchCreate + batchUpdate + handleDelete) |

## 6. Cell (row.get(slug))

| Property | R/W |
|---|---|
| `slug` | R |
| `value` | R/W (SCRIPT shape) |
| `readOnly` | R/W |
| `required` | R/W |
| `displayHtml` | R/W (cosmetic only — sanitize DOMPurify) |

## 7. Value shape theo fieldType (CRITICAL)

> ⚠️ **2 SHAPE KHÁC NHAU** — đừng nhầm:
> - **§7 dưới đây = SCRIPT shape** (cho `row.get(slug).value`, `field.value`, `$record.X`, `changedRowCells[i].newValue`, `screen.formItems[i].value`).
> - **§7B = API record shape** (cho `rl.setData(records)`). Khác hẳn — vd `date` cần `number` (epoch ms) thay vì `Date`, `lookup` chấp nhận cả object/string/array.

### Single (multiple=false)

| fieldType | Shape |
|---|---|
| `short_text`, `long_text`, `email`, `regex`, `phone`, `url` | `string` |
| `number`, `percent`, `currency`, `duration`, `auto_number` | `number` |
| `boolean` | `boolean` |
| `single_choice`, `radio_button` | `string` (slug của option) |
| `date`, `date_time` | `Date \| null` — dùng `new Date(...)` |
| `time` | `number \| null` — milliseconds từ 00:00. VD 09:30 = `(9*60+30)*60*1000` |
| `date_range`, `date_time_range` | `{ gte: Date \| null, lte: Date \| null }` |
| `time_range` | `{ gte: number \| null, lte: number \| null }` |
| `lookup_normal`, `reference`, `embedded`, `children` | **`string`** (chỉ ID của record, KHÔNG phải object) |
| `label`, `cascading` | `string` (slug/value) |
| `file` | `UploadFile` object (RHF array item đầu) |
| `lookup_parent`, `lookup_peer2peer`, `link` | RHF raw (fallthrough — hiếm dùng trong script) |
| `formula`, `rollup_summary` | string/number (read-only) |

### Multi (multiple=true)

| fieldType | Shape |
|---|---|
| text/number family (`short_text`, `number`, `email`, `phone`, `url`, ...) | `string[]` / `number[]` |
| `boolean` | `boolean[]` |
| `multi_choices`, `checkbox` | `string[]` |
| `date`, `date_time` | `(Date \| null)[]` |
| `time` | `(number \| null)[]` |
| `lookup_normal`, `reference`, `embedded`, `children` | **`string[]`** (mảng ID, KHÔNG phải mảng object) |
| `label`, `cascading` | `string[]` |
| `file` | `UploadFile[]` |

### ⚠️ Quan trọng — lookup KHÔNG có .name / .displayFields

Lookup field trong script value chỉ là **ID string**. Để đọc field khác của record được link, phải `filterRecords()` fetch lại:

```js
// SAI — $record.customer là string, không có .defaultAddress
const addr = $record.customer?.defaultAddress;   // → undefined

// ĐÚNG — fetch record bằng filterRecords
const customerId = $record.customer;
if (!customerId) return;
const { records } = await filterRecords("customer", {
    filterItems: [{ field: "id", op: "=", params: [customerId] }],
    limit: 1,
});
const addr = records[0]?.defaultAddress ?? "";
```

### Sai shape = render blank
```js
// SAI
row.get("date").value = "2026-05-26";              // → blank
row.get("time").value = "09:30";                   // → blank
row.get("lookup_field").value = { id: "abc-id" };  // → blank (chờ string)

// ĐÚNG
row.get("date").value = new Date("2026-05-26");
row.get("time").value = (9 * 60 + 30) * 60 * 1000;
row.get("lookup_field").value = "abc-id";          // chỉ cần ID string
```

## 7B. Value shape cho `rl.setData(records)` (KHÁC §7)

`rl.setData(records: RecordItem[])` chạy qua converter **API record shape → RHF shape**, KHÔNG dùng SCRIPT shape của §7. Mỗi record có dạng `{ id: string, [fieldSlug]: <API-shape-value> }`.

### Single (multiple=false)

| fieldType | Shape cho `record[slug]` trong setData | Lưu ý |
|---|---|---|
| `short_text`, `email`, `regex`, `label` | `string` | |
| `phone` | `string` | converter auto thêm `+` nếu thiếu |
| `name` (system) | `string` | |
| `long_text` | `{ value: string, text_type: 1 \| 2 }` hoặc plain `string` (legacy) | |
| `numeric`, `decimal`, `currency`, `auto_number` | `number` | |
| `percent` | `number` **ratio** (0.75 = 75%) — KHÔNG nhân 100 sẵn | converter sẽ x100 nếu `hasMulPercentValue=true` |
| `boolean`, `checkbox` (single) | `boolean` hoặc truthy/falsy (coerce `!!`) | |
| `single_choice`, `radio_button` | `string` (slug option) | |
| `date` | **`number` (epoch ms)** — vd `1776145500000`. ⚠️ ISO string `"2026-04-15"` → blank (default `convertDateToUTC=false`) | KHÁC §7 |
| `date_time` | **`number` (epoch ms)** | |
| `time` | `number` (ms-of-day từ 00:00) | giống §7 |
| `time_duration` | `number` (tổng ms) | |
| `date_range`, `date_time_range` | `{ gte, lte, start, end }` epoch ms (gte/start, lte/end fallback) | KHÁC §7 |
| `time_range` | `{ gte, lte, start, end }` ms-of-day | |
| `url` | `{ url: string, alias: string }` hoặc plain `string` (auto wrap) | |
| `lookup_normal`, `reference`, `embedded`, `children` | **`RecordItem` (object có `.id`) HOẶC `string` (id thẳng) HOẶC array của 2 dạng** | converter tự extract `.id` |
| `cascading` | `string[]` (luôn array, kể cả single = 1 leaf) | |
| `file` | object metadata `{ fileName, file_id, fileExt, url, ... }` (xem RULE-RECORD-06) | |
| `rating` | `number` | |
| `data_table` | `array` hoặc JSON string | |
| `formula`, `rollup_summary` | (read-only, BE tự compute) | bỏ qua khi setData |

### Multi (multiple=true)

| fieldType | Shape |
|---|---|
| text/number family (`short_text`, `email`, `phone`, `numeric`, ...) | mảng raw `string[]` / `number[]` (converter wrap mỗi item thành `[{data: ...}]` cho RHF) |
| `multi_choices`, `checkbox` | `string[]` (slug) |
| `date`, `date_time` | **`number[]`** (epoch ms array) |
| `time` | `number[]` (ms-of-day array) |
| `lookup_normal`, `reference`, `embedded`, `children` | mảng — mỗi item `RecordItem \| string` |
| `label` | `string[]` |
| `file` | array các object metadata |

### Ví dụ paste-ready

```js
// records từ filterRecords → API shape sẵn → pass thẳng cho setData, không map gì
const { records } = await filterRecords("order_line", {
    filterItems: [{ field: "order", op: "=", params: [orderId] }],
    limit: 500,
});
rl.setData(records);   // ✅ works — converter handle mọi field types

// Hoặc build record thủ công
rl.setData([
    {
        id: "tmp-1",                        // id required theo TS, runtime sẽ override bằng UUID
        product: "prod-uuid-1",             // lookup: pass string ID
        quantity: 5,
        price: 100000,
        delivery_date: 1776145500000,       // date: EPOCH MS (KHÔNG phải Date object)
        delivery_time: (14 * 60 + 30) * 60 * 1000,  // time: ms-of-day
        is_priority: true,
        tags: ["urgent", "fragile"],        // multi_choices: string[]
    },
]);
```

### Sai shape phổ biến

```js
// SAI — Date object cho date field qua setData
rl.setData([{ id: "tmp", delivery_date: new Date("2026-05-30") }]);
// → converter Number(Date object) = NaN → null → blank

// SAI — ISO string "YYYY-MM-DD" cho date
rl.setData([{ id: "tmp", delivery_date: "2026-05-30" }]);
// → Number("2026-05-30") = NaN → blank (vì convertDateToUTC=false default)

// ĐÚNG
rl.setData([{ id: "tmp", delivery_date: new Date("2026-05-30").getTime() }]);
// hoặc dayjs("2026-05-30").valueOf()
```

## 8. filterRecords

```js
const { records, total, meta } = await filterRecords("product", {
    filterItems: [
        { field: "status", op: "=", params: ["active"] },
        { field: "price", op: ">", params: [100000] },
    ],
    limit: 20,
});
```

> **Đã có cache built-in**: dùng `queryClient.fetchQuery` của React Query, queryKey = `[GET_LIST_RECORD, objectId, filterParams]`. Gọi nhiều lần với CÙNG `filterObjectSlug` + CÙNG `params` → cache hit, KHÔNG gọi API lại.
>
> → **KHÔNG wrap `$ref.cached = await filterRecords(...)`**. Code thừa + có thể giữ data stale. Cứ gọi thẳng `filterRecords` mỗi lần cần, cache lo phần network. Xem anti-pattern §13.

## 13. Anti-patterns

| Anti-pattern | Vì sao sai | Đúng |
|---|---|---|
| Wrap `$ref` cache quanh `filterRecords` | React Query đã cache (queryKey objectId+params). Self-cache giữ data stale. | Gọi thẳng `filterRecords` mỗi lần. |
| SET field cha không so sánh `if (item.value !== ...)` | Trigger re-render loop vì value changed → script re-run → set lại. | Luôn so sánh trước khi set. |
| `rl.setData(...)` không check `existingRows.length === 0` | Ghi đè data user nhập tay. | Guard `if (existingRows.length > 0) return;` trừ khi business yêu cầu overwrite. |
| Đọc `$record.<lookup>.<field>` để lấy data record được link | `$record.<lookup>` là string ID, không có `.field`. | `filterRecords(objectSlug, { filterItems: [{ field: "id", op: "=", params: [id] }] })`. |
| Dùng `c.newValue` mà nghĩ là RHF raw | Đã convert SCRIPT shape (từ SPT-2033). Cùng shape với `row.get(slug).value`. | Tin tưởng dùng thẳng. |
| Quên `await` cho `rl.row/rows/findRow/submit/filterRecords` | Nhận Promise thay vì kết quả → mọi check sau đó sai. | Luôn `await`. |

## 9. Async detection

Script tự wrap async nếu chứa `await`. **Dùng `await` cho mọi `rl.row/rows/findRow/submit/filterRecords`** — không sẽ nhận `Promise` thay vì kết quả.

## 10. Re-run trigger

Script chạy lại khi:
- Field cha thay đổi (`changedFields[slug]` = true)
- Cell related list thay đổi (`changedRowCells` buffer)
- Layout id đổi

Debounce 50ms — multiple changes liên tiếp → 1 exec.

## 11. Loop guard

Khi script SET cell value qua API → notify channel block tạm thời (200ms debounce) để tránh infinite loop. Nhưng SET field cha vẫn có thể trigger re-run nếu giá trị mới khác giá trị cũ — phải tự check `if (item.value !== newValue)` trong code.

## 12. waitReady timeout

`rl.rows/row/findRow/submit` đợi bảng mount + load. Timeout 10s → trả `null`/`[]`. Script vẫn tiếp tục, nhưng kết quả không có. Check `if (!row) return;` sau mỗi await.
