## Cách sử dụng TextField Component của ui-kit

Đường dẫn: `ui-kit/src/components/TextField`

Props: `label`, `error`, `helperText`, `color`, `fullWidth`, `startAdornment`, `endAdornment`, `inputClassName`, `multipleLine`, `rows` (default 4), `readOnly`, `htmlReadOnly`, `isFocused`, `hideValue`, `defaultExpand`, `resize` (default "none"), `hideMaxLength`, `inputStyle`, `enabledOverflowY`, `aiLoading`, `showBorderOnBlur` (default true), `helperTextProps`. Hỗ trợ `React.InputHTMLAttributes` và `ref`.

```tsx
// Basic usage
<TextField fullWidth label="Tên" error={!valid} helperText={!valid ? "Bắt buộc" : undefined} />

// Textarea với auto-expand
<TextField multipleLine defaultExpand rows={3} maxLength={500} fullWidth />

// Adornments
<TextField startAdornment={<SearchIcon />} endAdornment={() => loading ? <Spinner /> : <CheckIcon />} />

// Read-only modes
<TextField readOnly value="text" />          // renders as div
<TextField htmlReadOnly value={date} endAdornment={<CalendarIcon />} />  // keeps input UI

// Hide value (not password)
<TextField hideValue value={secretKey} />

// Style input (not container)
<TextField inputClassName={cx("text-right")} />
```

**DO:** Dùng `multipleLine` thay vì `<textarea>`, `readOnly` để hiển thị data, `htmlReadOnly` giữ input UI, `error`+`helperText` cho validation, `hideValue` để ẩn giá trị không phải password, `hideMaxLength` để ẩn counter, `defaultExpand` để tự động giãn.

**DON'T:** Tự render `<textarea>`, wrap icon bên ngoài thay vì dùng `startAdornment`/`endAdornment`, dùng `disabled` thay `readOnly`, tự style border lỗi, override input style qua `className` (dùng `inputClassName`).

| Trạng thái | Border | Background |
|------------|--------|------------|
| Normal | `border-divider-primary` | `bg-primary-light-98` |
| Focus | `border-info` | `bg-primary-light-96` |
| Error | `border-error` | `bg-primary-light-98` |
| Disabled | `border-primary-light-94` | `bg-primary-light-94` |
