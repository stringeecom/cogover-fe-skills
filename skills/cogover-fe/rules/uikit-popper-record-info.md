---
title: Cách sử dụng PopperRecordInfo Component của ui-kit
impact: HIGH
impactDescription: Sử dụng PopperRecordInfo sai dẫn đến việc popup thông tin bản ghi không hiển thị, thiếu callback điều hướng, hoặc layout hiển thị không đúng
tags: popper, record, popup, hover, form-builder, ui-kit
---

## Cách sử dụng PopperRecordInfo Component của ui-kit

Đường dẫn component: `ui-kit/src/components/PopperRecordInfo`

`PopperRecordInfo` là component wrapper xung quanh `Popper`, hiển thị popup thông tin chi tiết của một bản ghi (record) khi hover vào phần tử con. Popup sẽ tự động fetch dữ liệu record và hiển thị dưới dạng form chỉ đọc thông qua `FormBuilder`, kèm nút "Xem chi tiết".

Props riêng: `objectSlug` (bắt buộc), `recordId` (bắt buộc), `layoutId`, `width`, `height`, `onClickViewDetail`. Kế thừa tất cả props của `Popper` ngoại trừ `render` (đã được xử lý nội bộ).

---

### RULE-PRI-01: Luôn truyền `objectSlug` và `recordId` — đây là hai props bắt buộc

`objectSlug` dùng để xác định loại đối tượng (object), `recordId` dùng để fetch chi tiết bản ghi. Thiếu một trong hai sẽ khiến popup không hiển thị được dữ liệu.

**Sai:**

```tsx
// ❌ Thiếu objectSlug — popup không thể fetch được thông tin object
<PopperRecordInfo recordId={record.id}>
  {(params) => <span {...params}>{record.name}</span>}
</PopperRecordInfo>
```

**Đúng:**

```tsx
// ✅ Truyền đầy đủ objectSlug và recordId
<PopperRecordInfo objectSlug="customer" recordId={record.id}>
  {(params) => <span {...params}>{record.name}</span>}
</PopperRecordInfo>
```

---

### RULE-PRI-02: Sử dụng `onClickViewDetail` thay vì tự xử lý điều hướng

`PopperRecordInfo` đã có sẵn nút "Xem chi tiết" bên trong popup. Khi click, callback `onClickViewDetail` sẽ được gọi với `{ record, object }`. Sử dụng callback này để điều hướng thay vì tự thêm link hoặc button bên ngoài.

**Sai:**

```tsx
// ❌ Tự thêm nút điều hướng bên ngoài — duplicate logic đã có sẵn trong popup
<div>
  <PopperRecordInfo objectSlug="customer" recordId={record.id}>
    {(params) => <span {...params}>{record.name}</span>}
  </PopperRecordInfo>
  <button onClick={() => navigate(`/records/${record.id}`)}>Xem chi tiết</button>
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng onClickViewDetail để xử lý điều hướng khi click nút trong popup
<PopperRecordInfo
  objectSlug="customer"
  recordId={record.id}
  onClickViewDetail={({ record, object }) => {
    navigate(`/records/${object.slug}/${record.id}`);
  }}
>
  {(params) => <span {...params}>{record.name}</span>}
</PopperRecordInfo>
```

---

### RULE-PRI-03: Sử dụng `width` và `height` (số, đơn vị px) để tùy chỉnh kích thước popup

`PopperRecordInfo` sử dụng CSS variable `--width` và `--height` từ props `width` (mặc định 400) và `height` để đặt kích thước popup. Popup đã có sẵn `max-width: 90vw` và `max-height: calc(50svh - 46px)` để đảm bảo responsive. Không dùng className hoặc style để override kích thước.

**Sai:**

```tsx
// ❌ Override kích thước bằng className — gây xung đột với CSS variable nội bộ
<PopperRecordInfo
  objectSlug="customer"
  recordId={record.id}
  className="w-[500px]"
>
  {(params) => <span {...params}>{record.name}</span>}
</PopperRecordInfo>
```

**Đúng:**

```tsx
// ✅ Sử dụng prop width/height để tùy chỉnh kích thước
<PopperRecordInfo
  objectSlug="customer"
  recordId={record.id}
  width={500}
  height={300}
>
  {(params) => <span {...params}>{record.name}</span>}
</PopperRecordInfo>
```

---

### RULE-PRI-04: Sử dụng `layoutId` khi cần hiển thị layout cụ thể thay vì layout mặc định

Mặc định, `PopperRecordInfo` sẽ tự động lấy layout VIEW mặc định của object. Khi cần hiển thị theo layout cụ thể, truyền `layoutId`. Không tự fetch layout rồi render `FormBuilder` thủ công.

**Sai:**

```tsx
// ❌ Tự fetch layout và render FormBuilder — duplicate toàn bộ logic đã có sẵn
const { data: layout } = useGetLayout({ objectSlug: "customer", layoutId: "custom-layout" });

<Popper
  isShowOnHover
  render={({ attributes, ...params }) => (
    <div {...params} {...attributes}>
      <FormBuilder items={layout} detailForm canEdit={false} />
    </div>
  )}
>
  {(params) => <span {...params}>{record.name}</span>}
</Popper>
```

**Đúng:**

```tsx
// ✅ Truyền layoutId để PopperRecordInfo tự xử lý
<PopperRecordInfo
  objectSlug="customer"
  recordId={record.id}
  layoutId="custom-layout-id"
>
  {(params) => <span {...params}>{record.name}</span>}
</PopperRecordInfo>
```

---

### RULE-PRI-05: Không tự tạo Popper hiển thị thông tin record — sử dụng PopperRecordInfo

`PopperRecordInfo` đã xử lý: hover delay (1000ms show / 100ms hide), fetch object và record, render form chỉ đọc qua `FormBuilder`, nút xem chi tiết với link tự động cho các system objects (nhân sự, phòng ban, chức vụ), portal vào `document.body`, và ngăn sự kiện click lan ra ngoài. Không tự tạo lại các tính năng này.

**Sai:**

```tsx
// ❌ Tự tạo popper hiển thị record — thiếu nhiều tính năng đã có sẵn
<Popper
  isShowOnHover
  delay={[1000, 100]}
  appendTo={{ current: document.body }}
  render={({ attributes, ...params }) => (
    <div {...params} {...attributes} className="w-[400px] p-4 bg-white shadow rounded">
      <RecordDetail objectSlug="customer" recordId={record.id} />
      <button onClick={() => navigate(`/records/${record.id}`)}>Xem chi tiết</button>
    </div>
  )}
>
  {(params) => <span {...params}>{record.name}</span>}
</Popper>
```

**Đúng:**

```tsx
// ✅ Sử dụng PopperRecordInfo để có đầy đủ tính năng
<PopperRecordInfo
  objectSlug="customer"
  recordId={record.id}
  onClickViewDetail={({ record, object }) => {
    navigate(`/records/${object.slug}/${record.id}`);
  }}
>
  {(params) => <span {...params}>{record.name}</span>}
</PopperRecordInfo>
```

---

### RULE-PRI-06: Có thể truyền thêm các props của Popper (ngoại trừ `render`)

`PopperRecordInfo` kế thừa tất cả props của `Popper` ngoại trừ `render`. Có thể tùy chỉnh `placement`, `offset`, `delay`, `disabled`, và các props khác của Popper.

```tsx
// ✅ Tùy chỉnh placement và offset
<PopperRecordInfo
  objectSlug="customer"
  recordId={record.id}
  placement="top-start"
  offset={[0, 8]}
  disabled={!hasPermission}
>
  {(params) => <span {...params}>{record.name}</span>}
</PopperRecordInfo>
```

---

### RULE-PRI-07: Sử dụng render prop pattern của `children` giống như Popper

`PopperRecordInfo` sử dụng cùng render prop pattern cho `children` như `Popper`. Phần tử trigger phải nhận `ref` và các event handlers từ params.

**Sai:**

```tsx
// ❌ Không dùng render prop — popup sẽ không có anchor element
<PopperRecordInfo objectSlug="customer" recordId={record.id}>
  <span>{record.name}</span>
</PopperRecordInfo>
```

**Đúng:**

```tsx
// ✅ Sử dụng render prop pattern với params
<PopperRecordInfo objectSlug="customer" recordId={record.id}>
  {(params) => <span {...params}>{record.name}</span>}
</PopperRecordInfo>
```

---

## Giá trị mặc định

| Prop | Giá trị mặc định | Mô tả |
|------|-------------------|-------|
| `width` | `400` | Chiều rộng popup (px) |
| `height` | `undefined` | Chiều cao popup (px), tự động theo nội dung |
| `placement` | `bottom-start` | Vị trí popup so với trigger |
| `offset` | `[0, 4]` | Khoảng cách popup so với trigger (px) |
| `delay` | `[1000, 100]` | Delay show 1000ms, delay hide 100ms |
| `appendTo` | `document.body` | Portal popup vào body |
| `isShowOnHover` | `true` | Hiển thị popup khi hover |

## Cơ chế hoạt động

Thứ tự xử lý khi popup hiển thị:
1. Hover vào trigger element, sau 1000ms popup xuất hiện
2. Fetch thông tin object theo `objectSlug`
3. Fetch thông tin record theo `recordId` và `objectId`
4. Lấy layout VIEW (mặc định hoặc theo `layoutId`)
5. Render form chỉ đọc qua `FormBuilder`
6. Hiển thị nút "Xem chi tiết" — click sẽ gọi `onClickViewDetail` với `{ record, object }`
