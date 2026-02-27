## Cách sử dụng InputGroup Component của ui-kit

Đường dẫn: `ui-kit/src/components/InputGroup`

| Prop | Kiểu | Mặc định | Mô tả |
|------|------|----------|-------|
| `groupSelectProps` | `SelectProps<T>` | (bắt buộc) | Props cho Select bên trái (cần `value`, `onChange`, `options`) |
| `groupSelectWidth` | `number` | `108` | Chiều rộng (px) của Select |
| `fullWidth` | `boolean` | `false` | Chiếm toàn bộ chiều rộng container |
| `children` | `ReactNode` | — | Component input bên phải (phải có class `stringee-input-root` hoặc `stringee-tags-list`) |

Component tự động: loại bỏ bo góc phải của Select, loại bỏ bo góc trái của children, truyền `minWidthPopper="200"` và `fullWidth` cho Select nội bộ.

```tsx
// Cơ bản
<InputGroup
  groupSelectProps={{
    value: selectedOption,
    onChange: setSelectedOption,
    options: options,
    getLabel: (option) => option.label,
    getValue: (option) => option.value,
  }}
>
  <TextField value={text} onChange={handleChange} />
</InputGroup>

// Full width với select width tuỳ chỉnh
<InputGroup
  fullWidth
  groupSelectWidth={85}
  groupSelectProps={{ value: selectedType, onChange: setSelectedType, options: groupOptions }}
>
  <TextField />
</InputGroup>

// Children có thể là MultipleSelect, Select từ ui-kit
<InputGroup groupSelectProps={{ value, onChange, options }}>
  <MultipleSelect value={selectedValues} onChange={setSelectedValues} options={multiOptions} getLabel={o => o.label} getValue={o => o.value} />
</InputGroup>
```

**DO:** Truyền đầy đủ `value`, `onChange`, `options` trong `groupSelectProps`. Dùng `groupSelectWidth` thay vì CSS thủ công. Dùng `fullWidth` prop thay vì wrap trong `div className="w-full"`. Dùng ui-kit components làm children.

**DON'T:** Thiếu `onChange` hoặc `options` trong `groupSelectProps`. Truyền `groupSelectWidth={108}` hay `fullWidth={false}` (đã là mặc định). Dùng `<input>` HTML thuần làm children. Truyền lại `minWidthPopper` hay `fullWidth` trong `groupSelectProps`.
