## OptionTable Component (`uikit-option-table`)

Đường dẫn: `ui-kit/src/components/OptionTable`

Bộ 3 component dùng dựng bảng option/list nhẹ kiểu div-grid (không phải HTML `<table>`):

- `OptionTable` — wrapper `flex-col`, có `gap` (px) tuỳ chọn
- `OptionTableRow` — 1 dòng `flex items-center min-h-[2.5rem] rounded`, optional `isHeading` để áp `bg-gray-5`
- `OptionTableCell` — 1 ô, mặc định `flex={1}` co giãn; truyền `width` (px) để chốt độ rộng cố định; `isHeading` đổi typography `prose-body1` + `text-typo-secondary`

Props chính:

| Component | Prop | Mặc định | Ý nghĩa |
|---|---|---|---|
| `OptionTable` | `gap?: number` | `8` | px, áp vào `style.gap` |
| `OptionTable` | `className?` / `style?` / mọi `HTMLAttributes<HTMLDivElement>` | – | Pass-through xuống `<div>` |
| `OptionTableRow` | `gap?: number` | `8` | px, áp vào `style.gap` |
| `OptionTableRow` | `isHeading?: boolean` | `false` | `true` → áp `bg-gray-5` |
| `OptionTableRow` | `className?` / `style?` / mọi `HTMLAttributes<HTMLDivElement>` | – | Pass-through |
| `OptionTableCell` | `flex?: number` | `1` | Áp qua CSS var `--flex` → class `flex-[--flex]` |
| `OptionTableCell` | `width?: number` | – | Có giá trị → switch sang `w-[--width] flex-[unset]`, `flex` bị ignore |
| `OptionTableCell` | `isHeading?: boolean` | `false` | `true` → `text-typo-secondary !prose-body1` |

`OptionTable` và `OptionTableRow` đều `forwardRef<HTMLDivElement>`. `OptionTableCell` không forwardRef.

---

### RULE-OPTION-TABLE-01: Dùng cho danh sách option / bảng cấu hình ngắn — KHÔNG dùng thay `Table` / `ListViewTable`

`OptionTable` chỉ là grid div nhẹ, không có:
- Sticky header / sort / pin column
- Pagination, filter, search
- Lazy load, virtualization
- Selection state, row checkbox tích hợp

```tsx
// ❌ Bảng dữ liệu chính (records, danh sách entity) → dùng ListViewTable
<OptionTable>
    {records.map(...)}
</OptionTable>

// ✅ Mục đích đúng: bảng cấu hình ngắn, danh sách option, mapping pair, summary table
<OptionTable gap={8}>
    <OptionTableRow isHeading>
        <OptionTableCell isHeading flex={2}>Tên</OptionTableCell>
        <OptionTableCell isHeading width={120}>Trạng thái</OptionTableCell>
    </OptionTableRow>
    {options.map((opt) => (
        <OptionTableRow key={opt.id}>
            <OptionTableCell flex={2}>{opt.name}</OptionTableCell>
            <OptionTableCell width={120}>{opt.status}</OptionTableCell>
        </OptionTableRow>
    ))}
</OptionTable>
```

---

### RULE-OPTION-TABLE-02: Cell width lệch nhau → dùng `flex` + `width` đồng bộ giữa header và body

Mọi cell ở cùng index trong row cần cùng `flex` hoặc cùng `width` thì mới canh thẳng cột — không có cơ chế "column" tự động.

```tsx
// ❌ Header và body không cùng flex → cột lệch
<OptionTableRow isHeading>
    <OptionTableCell isHeading>Tên</OptionTableCell>
    <OptionTableCell isHeading>Trạng thái</OptionTableCell>
</OptionTableRow>
<OptionTableRow>
    <OptionTableCell flex={2}>{name}</OptionTableCell>
    <OptionTableCell width={120}>{status}</OptionTableCell>
</OptionTableRow>

// ✅ Đồng bộ flex/width giữa header và body
<OptionTableRow isHeading>
    <OptionTableCell isHeading flex={2}>Tên</OptionTableCell>
    <OptionTableCell isHeading width={120}>Trạng thái</OptionTableCell>
</OptionTableRow>
<OptionTableRow>
    <OptionTableCell flex={2}>{name}</OptionTableCell>
    <OptionTableCell width={120}>{status}</OptionTableCell>
</OptionTableRow>
```

---

### RULE-OPTION-TABLE-03: `isHeading` ở cấp Row vs cấp Cell — dùng cả 2 cho header thật sự

- `OptionTableRow isHeading` → bg `bg-gray-5` cho cả dòng
- `OptionTableCell isHeading` → đổi typography (`!prose-body1` + `text-typo-secondary`)

Hai prop độc lập. Dòng header thật sự cần truyền `isHeading` ở cả Row và mọi Cell trong dòng đó.

```tsx
// ✅ Header chuẩn
<OptionTableRow isHeading>
    <OptionTableCell isHeading flex={2}>Tên</OptionTableCell>
    <OptionTableCell isHeading width={120}>Trạng thái</OptionTableCell>
</OptionTableRow>

// ❌ Chỉ Row → text trông như body, không phân biệt header
<OptionTableRow isHeading>
    <OptionTableCell flex={2}>Tên</OptionTableCell>
</OptionTableRow>

// ❌ Chỉ Cell → typography đúng nhưng nền không nổi bật
<OptionTableRow>
    <OptionTableCell isHeading flex={2}>Tên</OptionTableCell>
</OptionTableRow>
```

---

### RULE-OPTION-TABLE-04: `gap` API là **px** — không truyền rem string

Cả `OptionTable` và `OptionTableRow` đều nhận `gap?: number` và áp `style.gap = '${n}px'`. Đây là exception duy nhất khỏi rule "spacing rem-only" vì API runtime đã cố định ở px.

```tsx
// ❌ Sai kiểu — TS error
<OptionTable gap="0.5rem" />

// ❌ Sai đơn vị — không có đơn vị
<OptionTable gap={0.5} />   // ra "0.5px" gần như vô hình

// ✅ Truyền số nguyên px
<OptionTable gap={8} />
<OptionTableRow gap={12} />
```

Nếu cần spacing rem ở trong row/cell → dùng `className` với `gap-[0.5rem]` thay vì prop.

---

### RULE-OPTION-TABLE-05: `width` cell là **px** — `flex` bị ignore khi có `width`

Khi `OptionTableCell` nhận `width`, class chuyển sang `w-[--width] flex-[unset]` — `flex` không còn tác dụng kể cả truyền vào.

```tsx
// ❌ Vô nghĩa: flex bị ignore vì có width
<OptionTableCell flex={2} width={120}>Status</OptionTableCell>

// ✅ Cell co giãn
<OptionTableCell flex={2}>Tên dài</OptionTableCell>

// ✅ Cell cố định
<OptionTableCell width={120}>Trạng thái</OptionTableCell>
```

---

### RULE-OPTION-TABLE-06: Custom style override được — `style.gap` và `style.<bất-kỳ>` thắng prop

Cả `OptionTable` và `OptionTableRow` build `style` cuối dạng `{ gap: '${gap}px', ...divProps.style }` → spread cuối → user truyền `style.gap` sẽ override prop `gap`.

```tsx
// gap prop = 8, nhưng style override → cuối cùng "32px"
<OptionTable gap={8} style={{ gap: '32px' }} />
```

Khi cần spacing động không tròn px (vd theo design token) → truyền qua `style.gap` (CSS string) thay vì prop `gap`.
