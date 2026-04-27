---
title: Kiến thức về Filter Record
impact: HIGH
impactDescription: Sai shape filters/params/logic_sequence là lỗi phổ biến gây BE trả rỗng, sai kết quả, hoặc reject. Mỗi field type có format params riêng theo từng operator
tags: filter, record, FilterGenerator, FilterRecordParams, operator, params, logic-sequence, ui-kit
---

## Kiến thức về Filter Record

### Khái niệm

Filter record cho phép truy vấn danh sách bản ghi của object theo điều kiện trên các field. Có 2 mặt:

- **API**: `useFilterRecord` / `useInfinityRecord` / `useLookupRecord` (`ui-kit/src/apis/record/record.api.ts`) — gửi `POST /api/v1/records` với body `FilterRecordParams`.
- **UI**: component `FilterGenerator` (`ui-kit/src/components/FilterGenerator/index.tsx`) — render UI cho user pick field/operator/value, output `FilterGeneratorItem[]` để feed vào API.

### RULE-FILTER-01: Request body — `FilterRecordParams`

Định nghĩa: `ui-kit/src/apis/record/record.type.ts:46-59`.

| Field | Type | Required | Mô tả |
|---|---|---|---|
| `object_type` | `string` | one-of | ID object (1 trong 2 phải có) |
| `object_slug` | `string` | one-of | Slug object |
| `filters` | `FilterGeneratorItem[]` | optional | Mảng condition (xem RULE-02) |
| `logic_sequence` | `string` | optional | Công thức logic, vd `"1 AND (2 OR 3)"` — index từ **1** |
| `type` | `FilterRecordTypeParam` | optional | `0=NONE`, `1=AND`, `2=OR`, `3=CUSTOM`. Auto-detect từ `logic_sequence` nếu không truyền |
| `size` | `number` | optional | Page size (default 10) |
| `search_after` | `string[]` | optional | Cursor pagination — opaque, copy nguyên từ response trước |
| `show_detail_on_record` | `boolean` | optional | Trả về full record details (object cho lookup, file metadata...) |
| `sorts` | `Record<string, {order:"asc"\|"desc"}>[]` | optional | Sort fields |
| `quick_search` | `string` | optional | Full-text search |
| `fields` | `string[]` | optional | Giới hạn field response trả về |
| `formula_fields` | `string[]` | optional | Computed fields (hỗ trợ dot notation) |

⚠️ Nếu `filters` rỗng, `record.api.ts:1198-1210` tự inject `[{field:"id", op:"not null", params:null}]` để tránh BE trả rỗng.

### RULE-FILTER-02: Item filter — `FilterGeneratorItem`

Định nghĩa: `ui-kit/src/components/FilterGenerator/type.ts:115-121`.

```typescript
interface FilterGeneratorItem {
  field: string | null;          // Slug field (có thể nested: "customer.id")
  op: Operator | null;           // Operator (xem RULE-03)
  params: FilterParamsType;      // Giá trị — shape phụ thuộc fieldType + op (xem RULE-04)
  fieldType?: DataTypes | null;  // short_text, numeric, date, lookup_normal...
  iu?: boolean;                  // Cờ "is current user" (cho field user-related)
}
```

**Field path conventions**:

| Field type | `field` của filter |
|---|---|
| `short_text`, `numeric`, các field thường | `<slug>` (vd `"name"`, `"amount"`) |
| `lookup_normal` | `<slug>` (vd `"customer"`) |
| `reference` (dependency) | `<slug>.id` (vd `"customer.id"` — bắt buộc suffix `.id`) |
| `cascading` nested | `<slug>.<level>.<...>` (vd `"department.level_2.slug"`) |

### RULE-FILTER-03: Operator (full enum)

Định nghĩa: `ui-kit/src/components/FilterGenerator/type.ts:51-95`.

| Nhóm | Operators |
|---|---|
| **So sánh** | `=`, `!=`, `<`, `>`, `<=`, `>=` |
| **Text matching** | `like`, `not like`, `startsWith`, `endsWith`, `contains any`, `not contains any` |
| **Set** | `in`, `not in`, `between`, `rangeWithin`, `rangeContains` |
| **Null check** | `is null`, `not null` |
| **Date relative** (params `null` hoặc `number` count) | `Yesterday`, `Today`, `Tomorrow`, `Last n days`, `Next n days`, `n Days ago`, `n Days from now`, `Last/This/Next week`, `Last/Next n weeks`, `Last/This/Next month`, `Last/Next n months`, `Last/This/Next quarter`, `Last/This/Next year`, `Last/Next n years` |

Operator cho từng field type được map qua `getConditionOptions(dataField)` tại `helpers.ts:448-550`. Tham khảo nhanh:

| Field type | Operators áp dụng |
|---|---|
| `short_text`, `email`, `phone`, `regex`, `auto_number`, `url` | `=`, `!=`, `like`, `not like`, `startsWith`, `endsWith`, `is null`, `not null`, optional `in`/`not in` |
| `long_text` | `contains any`, `not contains any`, `like`, `not like`, `is null`, `not null` |
| `numeric`, `decimal`, `percent`, `currency`, `time`, `time_duration` | `=`, `!=`, `<`, `>`, `<=`, `>=`, `between`, `is null`, `not null` |
| `date`, `date_time` | `=`, `!=`, `between`, `is null`, `not null`, full bộ date relative |
| `single_choice`, `radio_button` | `=`, `!=`, `in`, `not in`, `is null`, `not null` |
| `multi_choices`, `checkbox`, `label` | `in`, `not in`, `is null`, `not null` |
| `date_range`, `time_range`, `date_time_range` | `=`, `between`, `rangeWithin`, `rangeContains`, `is null`, `not null` |
| `rating` (star) | numeric ops + `between` + null check |
| `rating` (reaction) | `=`, `!=`, `in`, `not in`, null check |
| `cascading` | `=`, `!=` (single) hoặc `in`/`not in` (multiple), null check |
| `lookup_normal`, `embedded`, `reference`, `children` | `=`, `!=` (single) hoặc `in`/`not in` (multiple), null check |
| `file` | CHỈ `is null`, `not null` (không filter theo nội dung file) |

### RULE-FILTER-04: Format `params` theo field type × operator

Source of truth: `ui-kit/src/components/FilterGenerator/components/FilterItem/FilterParamsInput.tsx` (chuyển đổi UI value → API params).

| Field type | Operators | Shape `params` | Ví dụ |
|---|---|---|---|
| `short_text`, `email`, `phone`, `url`, `regex`, `auto_number` | `=`, `!=`, `like`, `not like`, `startsWith`, `endsWith` | `string` | `"abc"` |
| (cùng nhóm) | `in`, `not in` | `string[]` | `["a","b"]` |
| `long_text` | `contains any`, `not contains any`, `like`, `not like` | `string` | `"từ khoá"` |
| `numeric`, `decimal`, `currency`, `percent`, `time_duration` | `=`, `!=`, `<`, `>`, `<=`, `>=` | `number` | `100` |
| (cùng nhóm) | `between` | `[number, number]` | `[10, 50]` |
| `date` | `=`, `!=`, comparisons | ISO `"YYYY-MM-DD"` | `"2026-04-15"` |
| `date` | `between` | `[isoStart, isoEnd]` | `["2026-01-01","2026-12-31"]` |
| `date` / `date_time` | `Yesterday`, `Today`, `Tomorrow`, `Last week`, ... | `null` | `null` |
| `date` / `date_time` | `Last n days`, `Next n weeks`, `n Days ago`, ... | `number` (count) | `7` |
| `date_time` | `=`, `!=`, comparisons | epoch ms `number` hoặc ISO datetime | `1776145500000` |
| `time` | `=`, `!=`, comparisons, `between` | ms-of-day `number` | `44100000` (12:15 PM) |
| `date_range`, `time_range`, `date_time_range` | `=`, `between`, `rangeWithin`, `rangeContains` | `[start, end]` tuple | `["2026-01-01","2026-12-31"]` |
| `boolean` | `=`, `!=` | `1` (true) hoặc `0` (false) — **KHÔNG phải `true`/`false`** | `1` |
| `single_choice`, `radio_button` | `=`, `!=` | option slug `string` | `"option_1"` |
| (cùng nhóm) | `in`, `not in` | `string[]` slug | `["option_1","option_2"]` |
| `multi_choices`, `checkbox` | `in`, `not in` | `string[]` slug | `["a","b"]` |
| `label` | `in`, `not in` | `string[]` (kebab-case) | `["tag-1","tag-2"]` |
| `lookup_normal`, `embedded` | `=`, `!=` | record id `string` | `"REC123"` |
| (cùng nhóm, multiple) | `in`, `not in` | `string[]` ids | `["REC1","REC2"]` |
| `reference` (dependency) | `=`, `!=` (field path `<slug>.id`) | record id `string` | `"REC123"` |
| `cascading` | `=`, `!=` (single) | leaf slug `string` | `"leaf_a"` |
| `cascading` (multiple) | `in`, `not in` | `string[]` slugs | `["leaf_a","leaf_b"]` |
| `rating` (star) | numeric ops, `between` | `number` | `4` |
| `rating` (reaction) | `=`, `!=`, `in`, `not in` | reaction index `number` hoặc `number[]` | `3` |
| `file` | `is null`, `not null` (only) | `null` | — |
| **Mọi field** | `is null`, `not null` | `null` | `null` |

Helper convert date/time relative trước khi gửi: `helpers.ts:328-360` (`convertDateTimeInFilterItems()`).

### RULE-FILTER-05: Logic types & `logic_sequence`

`ui-kit/src/components/FilterGeneratorBase/type.ts:3`.

| `logicType` | `type` | `logic_sequence` | Mô tả |
|---|---|---|---|
| `"AND"` | `1` | `"1 AND 2 AND 3"` (auto từ count) | Tất cả conditions phải đúng |
| `"OR"` | `2` | `"1 OR 2 OR 3"` (auto) | Bất kỳ condition đúng |
| `"CUSTOM"` | `3` | `"1 AND (2 OR 3)"` (user viết) | Logic tự do |
| `"NONE"` | `0` | `""` | Không filter |

⚠️ **Index trong `logic_sequence` bắt đầu từ 1** (vị trí condition trong `filters[]` cộng 1). Sai index → BE reject. Validate qua `validateExpression(logicValue, itemCount)` trong `FilterGeneratorBase/helpers`.

### RULE-FILTER-06: Ví dụ payload thực tế

**1) Tên chứa "abc" VÀ amount > 100**

```typescript
{
  object_slug: "customer",
  filters: [
    { field: "name", op: "like", params: "abc", fieldType: "short_text" },
    { field: "amount", op: ">", params: 100, fieldType: "numeric" },
  ],
  logic_sequence: "1 AND 2",
  type: 1,
  size: 20,
}
```

**2) Filter related list (reference dependency) cho 1 customer cụ thể**

```typescript
{
  object_slug: "order",
  filters: [{ field: "customer.id", op: "=", params: "CUS-123", fieldType: "reference" }],
  logic_sequence: "1",
  type: 1,
}
```

**3) Custom logic — `(status=active OR vip=true) AND created trong 7 ngày qua`**

```typescript
{
  object_slug: "lead",
  filters: [
    { field: "status",  op: "=",          params: "active", fieldType: "single_choice" },
    { field: "vip",     op: "=",          params: 1,        fieldType: "boolean" },
    { field: "created", op: "Last n days", params: 7,        fieldType: "date_time" },
  ],
  logic_sequence: "(1 OR 2) AND 3",
  type: 3,
}
```

**4) Pagination với `search_after`**

```typescript
// Page 1
{ object_slug: "customer", filters: [...], size: 50, search_after: [] }
// Response trả về search_after: ["2026-04-27", "REC789"]

// Page 2 — copy nguyên cursor
{ object_slug: "customer", filters: [...], size: 50, search_after: ["2026-04-27", "REC789"] }
```

### RULE-FILTER-07: Component `FilterGenerator` (build UI filter)

Entry: `ui-kit/src/components/FilterGenerator/index.tsx`. Component **controlled** — parent quản lý state qua `react-hook-form`.

**Props chính** (`type.ts:15-44`):

| Prop | Type | Vai trò |
|---|---|---|
| `dataFields` | `DataField[]` hoặc `SelectGroupOption<DataField>[]` | Danh sách field cho user chọn filter |
| `filterItems` | `FilterGeneratorItem[]` | Mảng condition hiện tại |
| `onChangeFilterItems` | `(items) => void` | Callback khi thêm/sửa/xoá condition |
| `logicType` | `"AND" \| "OR" \| "CUSTOM" \| "NONE"` | Mode kết hợp |
| `logicValue` | `string` | Công thức logic khi `logicType="CUSTOM"` |
| `onChangeLogicType` / `onChangeLogicValue` | callbacks | |
| `disabled`, `readOnly`, `required` | boolean | Trạng thái UI |
| `maxFilterItemCount` | `number` | Giới hạn số condition (default 30) |
| `renderValueInputWrapper` | `(node, item) => ReactNode` | Wrap value input để custom validation/style |
| `renderLabel` | `(field) => ReactNode` | Custom render label combobox field |
| `showCustomLogicError` | `boolean` | Hiển thị lỗi parsing công thức custom |
| `showPreview` + `onClickPreview` | — | Nút "Xem trước" |
| `clearable` + `onReset` | — | Nút reset |
| `isWorkflow`, `includeAllFormulaFields` | boolean flags | Mode workflow / include formula fields |

**Cấu trúc nội bộ**:

- `FilterGenerator` → render N × `FilterItem`
- `FilterItem` → 3 phần: combobox chọn field + dropdown operator + `FilterParamsInput`
- `FilterParamsInput` → switch theo `fieldType` để render đúng input UI:
  - `short_text`/`email`/`phone` → `TextField`
  - `numeric`/`decimal` → `InputNumber`
  - `date`/`date_time` → `DatePicker` (có relative presets)
  - `date_range` → `DateRangePicker`
  - `time_duration` → `DurationInput`/`DurationRangeInput`
  - `single_choice` → `Select`
  - `multi_choices`/`checkbox` → `MultipleSelect`
  - `embedded`/`reference`/`lookup_normal` → `RecordMultipleSelect`
  - `cascading` → `TreeSelect`
  - `rating` → `InputNumber` (star) hoặc `Select` (reaction)
  - `file` → ẩn input (chỉ filter null check)
- Operator dropdown lấy từ `getConditionOptions(field)` tại `helpers.ts:448` — auto map field type → list operators hợp lệ.

**Sort/sortFields**: KHÔNG nằm trong `FilterGenerator`. Component riêng `FilterDataFieldSort` — dùng kèm độc lập.

### RULE-FILTER-08: Usage pattern với `react-hook-form`

Tham khảo: `ui-kit/src/components/RelatedListTable/RecordTable/ModalAddFromList/PopperFilter/index.tsx:28-130`.

```tsx
const methods = useForm<{ filterItems: FilterGeneratorItem[]; logicType: FilterGeneratorLogicTypes; logicValue: string }>({
  defaultValues: { filterItems: [], logicType: "AND", logicValue: "" }
});

<FormProvider {...methods}>
  <Controller name="filterItems" control={methods.control} render={({ field: f1 }) => (
    <Controller name="logicType" control={methods.control} render={({ field: f2 }) => (
      <Controller name="logicValue" control={methods.control} render={({ field: f3 }) => (
        <FilterGenerator
          dataFields={dataFields}
          filterItems={f1.value} onChangeFilterItems={f1.onChange}
          logicType={f2.value} onChangeLogicType={f2.onChange}
          logicValue={f3.value} onChangeLogicValue={f3.onChange}
        />
      )} />
    )} />
  )} />
</FormProvider>
```

Sau khi user submit, map state form sang `FilterRecordParams` và gọi `useFilterRecord`:

```typescript
const { data } = useFilterRecord({
  object_slug: "customer",
  filters: methods.getValues("filterItems"),
  logic_sequence: methods.getValues("logicValue") || autoBuildLogic(filterItems.length, logicType),
  type: logicTypeToTypeParam(logicType),
  size: 20,
});
```

### RULE-FILTER-09: Pitfalls dễ sai

1. **Index `logic_sequence` từ 1**, không phải 0. `"1 AND 2"` chứ không `"0 AND 1"`.
2. **`reference` field path bắt buộc suffix `.id`**: `field: "customer.id"`. `lookup_normal` thì ngược lại — KHÔNG suffix (`field: "customer"`).
3. **Date relative ops format params đặc biệt**:
   - `"Today"`, `"Yesterday"`, `"Last week"`... → `params: null`
   - `"Last n days"`, `"Next n weeks"`... → `params: <number count>` (chỉ là số nguyên)
4. **`boolean` dùng `1`/`0`**, không phải `true`/`false` trong `params` của filter.
5. **Empty `filters` → BE auto inject `{field:"id", op:"not null", params:null}`** để không trả rỗng.
6. **`search_after` opaque cursor** — luôn copy y nguyên từ response trước, KHÔNG tự construct hay edit.
7. **`useFilterRecord` vs `useInfinityRecord` cùng API endpoint** — chỉ khác cách handle pagination (single page vs infinite scroll). Cùng `FilterRecordParams`.
8. **`file` field chỉ filter được null/not null** — không filter theo nội dung/tên file.
9. **`size > 100` có thể bị BE reject** — verify với BE config trước khi set giá trị lớn.
10. **Khi filter related list của object B (qua field lookup từ A → B), object_slug là A** (xem reference `data-model/related-list` RULE-RL-03/04).
