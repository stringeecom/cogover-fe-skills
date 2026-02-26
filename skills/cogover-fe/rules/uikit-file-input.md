---
title: Cách sử dụng FileInput Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng FileInput sai dẫn đến việc tạo input file thủ công dư thừa, không hiển thị tooltip hướng dẫn và xử lý trạng thái isMaxFile không đúng
tags: file-input, upload, file, drag-drop, ui-kit
---

## Cách sử dụng FileInput Component của ui-kit

Đường dẫn component: `ui-kit/src/components/FileInput`

Props: `isMaxFile`, `maxSize`, `canEdit`, `tooltip`, `disabled`, `tabIndex`, `acceptTypes`, `onChange`, `multipleFile`, `dataTestId`, `iconRight`, `className`. Hỗ trợ `ref` forwarding.

---

### RULE-FILEINPUT-01: Sử dụng `FileInput` thay vì tự tạo input file với nút chọn file

FileInput đã bao gồm sẵn vùng kéo thả (drop zone) với icon upload, label text, nút "Chọn file", input ẩn và Tooltip hiển thị thông tin định dạng + dung lượng cho phép. Không tự tạo lại các thành phần này.

**Sai:**

```tsx
// ❌ Tự tạo input file thủ công — FileInput đã xử lý tất cả
<div className={cx("border border-dashed", "p-[1rem]")}>
  <input type="file" onChange={handleChange} accept=".pdf,.docx" />
  <Button onClick={() => inputRef.current?.click()}>Chọn file</Button>
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng FileInput với đầy đủ props
<FileInput
  isMaxFile={false}
  canEdit={true}
  acceptTypes={["pdf", "docx"]}
  onChange={handleChange}
/>
```

---

### RULE-FILEINPUT-02: Luôn truyền `isMaxFile` để kiểm soát hiển thị component

Khi `isMaxFile={true}`, component sẽ return `null` (không render gì). Sử dụng prop này để ẩn FileInput khi đã đạt số lượng file tối đa. Không tự kiểm tra điều kiện bên ngoài rồi conditional render.

**Sai:**

```tsx
// ❌ Kiểm tra điều kiện bên ngoài là dư thừa — FileInput đã xử lý nội bộ
{!isMaxFile && (
  <FileInput
    isMaxFile={false}
    canEdit={canEdit}
    onChange={handleChange}
  />
)}
```

**Đúng:**

```tsx
// ✅ Truyền isMaxFile trực tiếp, component tự xử lý ẩn/hiện
<FileInput
  isMaxFile={isMaxFile}
  canEdit={canEdit}
  onChange={handleChange}
/>
```

---

### RULE-FILEINPUT-03: Truyền `tooltip` và `maxSize` để hiển thị thông tin hướng dẫn trong Tooltip

`tooltip` hiển thị danh sách định dạng file cho phép. `maxSize` (đơn vị bytes) hiển thị dung lượng tối đa. Cả hai đều hiển thị trong Tooltip khi hover vào vùng FileInput. Chỉ truyền khi có giá trị cần hiển thị.

**Sai:**

```tsx
// ❌ Tự wrap Tooltip bên ngoài — FileInput đã có Tooltip nội bộ
<Tooltip content="Cho phép: PDF, DOCX. Tối đa 5MB">
  <FileInput
    isMaxFile={false}
    canEdit={true}
    onChange={handleChange}
  />
</Tooltip>
```

**Đúng:**

```tsx
// ✅ Truyền tooltip và maxSize để FileInput tự hiển thị trong Tooltip nội bộ
<FileInput
  isMaxFile={false}
  canEdit={true}
  tooltip="PDF, DOCX, PNG"
  maxSize={5242880}
  onChange={handleChange}
/>
```

---

### RULE-FILEINPUT-04: Truyền `acceptTypes` dưới dạng mảng tên phần mở rộng (không có dấu chấm)

`acceptTypes` nhận mảng chuỗi phần mở rộng file. Component sẽ tự thêm dấu chấm (`.`) phía trước và nối bằng dấu phẩy để tạo giá trị `accept` cho input.

**Sai:**

```tsx
// ❌ Truyền phần mở rộng có dấu chấm — sẽ bị thành ".." khi render
<FileInput
  isMaxFile={false}
  canEdit={true}
  acceptTypes={[".pdf", ".docx"]}
  onChange={handleChange}
/>

// ❌ Truyền MIME type — component không xử lý MIME type
<FileInput
  isMaxFile={false}
  canEdit={true}
  acceptTypes={["application/pdf", "image/png"]}
  onChange={handleChange}
/>
```

**Đúng:**

```tsx
// ✅ Truyền tên phần mở rộng không có dấu chấm
<FileInput
  isMaxFile={false}
  canEdit={true}
  acceptTypes={["pdf", "docx", "png", "jpg"]}
  onChange={handleChange}
/>
```

---

### RULE-FILEINPUT-05: Sử dụng `multipleFile` khi cần cho phép chọn nhiều file cùng lúc

Khi `multipleFile={true}`, input sẽ cho phép chọn nhiều file và label text sẽ hiển thị dạng số nhiều. Mặc định chỉ cho phép chọn 1 file.

**Sai:**

```tsx
// ❌ Không truyền multipleFile nhưng xử lý nhiều file bên ngoài
<FileInput
  isMaxFile={false}
  canEdit={true}
  onChange={(e) => {
    // Sẽ chỉ nhận được 1 file vì input không có multiple
    const files = Array.from(e.target.files ?? []);
    handleMultipleFiles(files);
  }}
/>
```

**Đúng:**

```tsx
// ✅ Truyền multipleFile khi cần chọn nhiều file
<FileInput
  isMaxFile={false}
  canEdit={true}
  multipleFile={true}
  onChange={(e) => {
    const files = Array.from(e.target.files ?? []);
    handleMultipleFiles(files);
  }}
/>
```

---

### RULE-FILEINPUT-06: Sử dụng `iconRight` để thêm nội dung bên phải nút "Chọn file"

`iconRight` render bên phải nút "Chọn file", cùng hàng và cùng nhóm flex. Sử dụng prop này khi cần thêm icon hoặc nút bổ sung bên cạnh nút chọn file.

**Sai:**

```tsx
// ❌ Wrap FileInput trong div để thêm icon — phá vỡ layout nội bộ
<div className={cx("flex items-center gap-[0.5rem]")}>
  <FileInput
    isMaxFile={false}
    canEdit={true}
    onChange={handleChange}
  />
  <SmartPasteIcon />
</div>
```

**Đúng:**

```tsx
// ✅ Sử dụng iconRight để thêm nội dung bên phải nút chọn file
<FileInput
  isMaxFile={false}
  canEdit={true}
  iconRight={<SmartPasteButton onClick={handleSmartPaste} />}
  onChange={handleChange}
/>
```

---

### RULE-FILEINPUT-07: Reset giá trị input sau khi xử lý onChange

Sau khi xử lý file trong `onChange`, cần reset `e.target.value = ""` để cho phép chọn lại cùng file. FileInput không tự reset giá trị input.

**Sai:**

```tsx
// ❌ Không reset — không thể chọn lại cùng file
<FileInput
  isMaxFile={false}
  canEdit={true}
  onChange={(e) => {
    const files = Array.from(e.target.files ?? []);
    processFiles(files);
  }}
/>
```

**Đúng:**

```tsx
// ✅ Reset e.target.value sau khi xử lý
<FileInput
  isMaxFile={false}
  canEdit={true}
  onChange={(e) => {
    const files = Array.from(e.target.files ?? []);
    processFiles(files);
    e.target.value = "";
  }}
/>
```

---

## Tham chiếu Props

| Prop | Kiểu | Bắt buộc | Mặc định | Mô tả |
|------|------|----------|----------|-------|
| `isMaxFile` | `boolean` | Có | - | Khi `true`, component không render |
| `canEdit` | `boolean` | Có | - | Cho phép chỉnh sửa |
| `onChange` | `(e: ChangeEvent<HTMLInputElement>) => void` | Có | - | Callback khi chọn file |
| `maxSize` | `number` | Không | - | Dung lượng tối đa (bytes), hiển thị trong Tooltip |
| `tooltip` | `string` | Không | - | Danh sách định dạng cho phép, hiển thị trong Tooltip |
| `disabled` | `boolean` | Không | - | Vô hiệu hóa nút chọn file |
| `tabIndex` | `number` | Không | - | Tab index cho nút chọn file |
| `acceptTypes` | `string[]` | Không | `[]` | Mảng phần mở rộng file (không có dấu chấm) |
| `multipleFile` | `boolean` | Không | - | Cho phép chọn nhiều file |
| `dataTestId` | `string` | Không | - | Attribute `data-testid` cho input |
| `iconRight` | `ReactNode` | Không | - | Nội dung hiển thị bên phải nút chọn file |
| `className` | `string` | Không | - | Class tùy chỉnh cho container |

## Tham chiếu style

| Phần tử | Style |
|---------|-------|
| Container | `border-[2px] border-divider-primary border-dashed rounded`, `px-[1rem] py-[1.25rem]` |
| Label text | `prose-caption2 text-typo-secondary` |
| Nút chọn file | `Button variant="gray"` |
| Tooltip content | `prose-caption2 text-[#fff]` |
