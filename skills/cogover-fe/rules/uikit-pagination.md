---
title: Cách sử dụng Pagination Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng Pagination sai dẫn đến việc tự xử lý logic prev/next thủ công, text hiển thị sai định dạng, và perPage options không nhất quán
tags: pagination, paging, per-page, next, prev, ui-kit
---

## Cách sử dụng Pagination Component của ui-kit

Đường dẫn component: `ui-kit/src/components/Pagination`

Props: `total`, `currentPage`, `text`, `perPage`, `perPageOptions`, `disabledPrev`, `disabledNext`, `disabledPerPage`, `hideNextPrev`, `hidePerPage`, `onChangePerPage`, `onNextPage`, `onPrevPage`. Hỗ trợ `ref` forwarding.

---

### RULE-PAGINATION-01: Sử dụng `disabledPrev`/`disabledNext` chỉ khi cần override logic mặc định

Pagination tự động tính toán trạng thái disabled cho nút prev (khi `currentPage === 1`) và nút next (khi đã ở trang cuối). Chỉ truyền `disabledPrev`/`disabledNext` khi cần override logic mặc định này.

**Sai:**

```tsx
// ❌ Thủ công tính toán lại logic mà component đã xử lý
<Pagination
  total={100}
  currentPage={page}
  disabledPrev={page === 1}
  disabledNext={page * 20 >= 100}
  onPrevPage={handlePrev}
  onNextPage={handleNext}
/>
```

**Đúng:**

```tsx
// ✅ Để component tự xử lý logic disabled
<Pagination
  total={100}
  currentPage={page}
  onPrevPage={handlePrev}
  onNextPage={handleNext}
/>

// ✅ Chỉ truyền khi cần override (ví dụ: disabled khi đang loading)
<Pagination
  total={100}
  currentPage={page}
  disabledPrev={isLoading}
  disabledNext={isLoading}
  onPrevPage={handlePrev}
  onNextPage={handleNext}
/>
```

---

### RULE-PAGINATION-02: Tuỳ chỉnh text hiển thị qua prop `text`, không tự render text bên ngoài

Pagination hỗ trợ tuỳ chỉnh text hiển thị qua prop `text` với 2 trường: `display` (nhãn của phần per page, mặc định `"Display"`) và `showing` (text hiển thị vị trí bản ghi, mặc định `"Showing {{from}}-{{to}} of {{total}}"`). Sử dụng placeholder `{{from}}`, `{{to}}`, `{{total}}` để component tự động thay thế giá trị.

**Sai:**

```tsx
// ❌ Tự render text bên ngoài rồi ẩn text của component
<div className={cx("flex items-center gap-[0.5rem]")}>
  <span>Hiển thị {from}-{to} / {total}</span>
  <Pagination
    total={100}
    currentPage={page}
    hideNextPrev
    onChangePerPage={handleChangePerPage}
  />
</div>
```

**Đúng:**

```tsx
// ✅ Truyền text tuỳ chỉnh qua prop text
<Pagination
  total={100}
  currentPage={page}
  text={{
    display: "Hiển thị",
    showing: "Hiển thị {{from}}-{{to}} / {{total}}",
  }}
  onPrevPage={handlePrev}
  onNextPage={handleNext}
  onChangePerPage={handleChangePerPage}
/>
```

---

### RULE-PAGINATION-03: Sử dụng `perPageOptions` để tuỳ chỉnh danh sách lựa chọn, không tự tạo button group

Mặc định, Pagination hiển thị các tuỳ chọn `[20, 50, 100]`. Khi cần thay đổi danh sách này, sử dụng prop `perPageOptions`. Không tự tạo button group bên ngoài.

**Sai:**

```tsx
// ❌ Tự tạo button group cho per page
<div className={cx("flex items-center gap-[0.5rem]")}>
  {[10, 25, 50].map((size) => (
    <button key={size} onClick={() => setPerPage(size)}>
      {size}
    </button>
  ))}
  <Pagination total={100} currentPage={page} hidePerPage />
</div>
```

**Đúng:**

```tsx
// ✅ Truyền perPageOptions và perPage
<Pagination
  total={100}
  currentPage={page}
  perPage={perPage}
  perPageOptions={[10, 25, 50]}
  onChangePerPage={handleChangePerPage}
  onPrevPage={handlePrev}
  onNextPage={handleNext}
/>
```

---

### RULE-PAGINATION-04: Sử dụng `hideNextPrev` và `hidePerPage` để ẩn phần tử, không dùng CSS

Khi chỉ cần hiển thị một phần của Pagination (chỉ có phần chọn số bản ghi hoặc chỉ có nút prev/next), sử dụng prop `hideNextPrev` hoặc `hidePerPage`. Không dùng CSS để ẩn.

**Sai:**

```tsx
// ❌ Dùng CSS để ẩn phần per page
<Pagination
  total={100}
  currentPage={page}
  className="[&>.stringee-pagination-per-page]:hidden"
  onPrevPage={handlePrev}
  onNextPage={handleNext}
/>
```

**Đúng:**

```tsx
// ✅ Chỉ hiển thị nút prev/next, ẩn phần chọn per page
<Pagination
  total={100}
  currentPage={page}
  hidePerPage
  onPrevPage={handlePrev}
  onNextPage={handleNext}
/>

// ✅ Chỉ hiển thị phần chọn per page, ẩn nút prev/next
<Pagination
  total={100}
  currentPage={page}
  hideNextPrev
  perPage={perPage}
  onChangePerPage={handleChangePerPage}
/>
```

---

### RULE-PAGINATION-05: Luôn truyền `perPage` khi sử dụng `onChangePerPage`

Khi cho phép người dùng thay đổi số bản ghi mỗi trang, phải truyền cả `perPage` (giá trị hiện tại) và `onChangePerPage` (callback). Nếu không truyền `perPage`, component sẽ mặc định là `20` và không highlight đúng nút đang chọn.

**Sai:**

```tsx
// ❌ Thiếu perPage — không highlight đúng nút đang chọn
<Pagination
  total={100}
  currentPage={page}
  onChangePerPage={handleChangePerPage}
  onPrevPage={handlePrev}
  onNextPage={handleNext}
/>
```

**Đúng:**

```tsx
// ✅ Truyền cả perPage và onChangePerPage
<Pagination
  total={100}
  currentPage={page}
  perPage={perPage}
  onChangePerPage={handleChangePerPage}
  onPrevPage={handlePrev}
  onNextPage={handleNext}
/>
```

---

## Tham chiếu props

| Prop | Type | Mặc định | Mô tả |
|------|------|----------|-------|
| `total` | `number` | *(bắt buộc)* | Tổng số bản ghi |
| `currentPage` | `number` | *(bắt buộc)* | Trang hiện tại |
| `text` | `{ display?: string; showing?: string }` | `{ display: "Display", showing: "Showing {{from}}-{{to}} of {{total}}" }` | Tuỳ chỉnh text hiển thị |
| `perPage` | `number` | `20` | Số bản ghi mỗi trang |
| `perPageOptions` | `number[]` | `[20, 50, 100]` | Danh sách tuỳ chọn số bản ghi |
| `disabledPrev` | `boolean` | Tự động tính | Override trạng thái disabled nút prev |
| `disabledNext` | `boolean` | Tự động tính | Override trạng thái disabled nút next |
| `disabledPerPage` | `boolean` | `undefined` | Vô hiệu hoá các nút chọn per page |
| `hideNextPrev` | `boolean` | `undefined` | Ẩn phần nút prev/next và text showing |
| `hidePerPage` | `boolean` | `undefined` | Ẩn phần chọn số bản ghi mỗi trang |
| `onChangePerPage` | `(perPage: number) => void` | `undefined` | Callback khi thay đổi số bản ghi |
| `onNextPage` | `() => void` | `undefined` | Callback khi bấm nút next |
| `onPrevPage` | `() => void` | `undefined` | Callback khi bấm nút prev |
