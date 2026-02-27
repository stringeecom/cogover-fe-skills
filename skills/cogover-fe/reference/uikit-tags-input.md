## Cách sử dụng TagsInput Component của ui-kit

Path: `ui-kit/src/components/TagsInput`

Generic: `TagsInput<T>` — T defaults to string.

Key props: `value`, `onChange`, `defaultValue`, `fullWidth`, `error`, `helpText`, `color`, `tagVariants`, `tagProps`, `placeholder`, `clearable`, `onClearable`, `submitWithStringValue`, `isChangeOverlap`, `startAdornment`, `endAdornment`, `searchable`, `filterBySearch`, `suggestListOptions`, `renderSuggestItem`, `renderTag`, `renderLimitTag`, `renderText`, `getLabel`, `getValue`, `getOptionWidth`, `getInputRef`, `onInputChange`, `handleSuggestOptionClickExtend`, `handleDeleteOptionExtend`, `wrapperClassName`, `popperClassName`, `appendToBody`, `tabIndex`.

---

### Usage

```tsx
// Basic string tags
const [tags, setTags] = useState<string[]>([]);
<TagsInput value={tags} onChange={setTags} />

// Object tags — must provide getLabel and getValue
<TagsInput<User>
  value={selectedUsers}
  onChange={setSelectedUsers}
  getLabel={(user) => user.name}
  getValue={(user) => user.id}
/>

// Suggest list — submitWithStringValue={false} to restrict free text entry
<TagsInput
  submitWithStringValue={false}
  searchable
  filterBySearch
  suggestListOptions={options}
  renderSuggestItem={(option) => <PopperMenuItem>{option}</PopperMenuItem>}
  value={tags}
  onChange={setTags}
/>

// With clear button + side effect handling
<TagsInput
  clearable
  onClearable={(currentValues) => trackEvent("tags_cleared", { count: currentValues.length })}
  value={tags}
  onChange={setTags}
/>

// Custom tag rendering
<TagsInput
  renderTag={({ value, onDelete, isFocusedTag, disabled }) => (
    <Tag variant="outlined" isFocused={isFocusedTag} onDelete={!disabled ? onDelete : undefined} showDeleteIcon>
      {getLabel(value)}
    </Tag>
  )}
  value={tags}
  onChange={setTags}
/>

// In container with overflow hidden — appendToBody={false}
<div className="relative">
  <TagsInput appendToBody={false} suggestListOptions={options} renderSuggestItem={(o) => <span>{o}</span>} value={tags} onChange={setTags} />
</div>
```

---

### DO / DON'T

```tsx
// ❌ Object without getLabel/getValue — shows "[object Object]"
<TagsInput<User> value={selectedUsers} onChange={setSelectedUsers} />

// ❌ searchable without filterBySearch — list not filtered
<TagsInput searchable suggestListOptions={options} />

// ❌ filterBySearch without searchable — no effect
<TagsInput filterBySearch suggestListOptions={options} />

// ❌ Redundant — error auto-sets color="error"
<TagsInput error color="error" helpText="Required" value={tags} onChange={setTags} />

// ✅ Just error is enough
<TagsInput error helpText="Required" value={tags} onChange={setTags} />
```
