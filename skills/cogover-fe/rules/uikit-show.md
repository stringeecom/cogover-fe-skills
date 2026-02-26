---
title: Cách sử dụng Show Component của ui-kit
impact: MEDIUM
impactDescription: Sử dụng Show sai dẫn đến việc render không mong muốn, mất type safety khi dùng render function, và viết conditional rendering dài dòng không cần thiết
tags: show, conditional, rendering, when, fallback, ui-kit
---

## Cách sử dụng Show Component của ui-kit

Đường dẫn component: `ui-kit/src/components/Show`

Props: `when`, `children`, `fallback`. Hỗ trợ generic type `<T>` cho giá trị `when`.

---

### RULE-SHOW-01: Sử dụng `Show` thay vì toán tử `&&` hoặc ternary cho conditional rendering

`Show` giúp code khai báo (declarative) hơn, dễ đọc hơn và tránh lỗi render giá trị falsy (ví dụ: `0`, `""` bị render ra DOM khi dùng `&&`).

**Sai:**

```tsx
// ❌ Toán tử && có thể render giá trị falsy (0, "") ra DOM
{items.length && <ItemList items={items} />}

// ❌ Ternary dài dòng khi không có fallback
{isVisible ? <Content /> : null}
```

**Đúng:**

```tsx
// ✅ Sử dụng Show để conditional rendering rõ ràng
<Show when={items.length}>
  <ItemList items={items} />
</Show>

// ✅ Đơn giản và dễ đọc
<Show when={isVisible}>
  <Content />
</Show>
```

---

### RULE-SHOW-02: Sử dụng prop `fallback` thay vì đặt `Show` trong ternary

Khi cần hiển thị nội dung thay thế khi điều kiện không thỏa mãn, sử dụng prop `fallback` thay vì wrap `Show` trong biểu thức ternary.

**Sai:**

```tsx
// ❌ Ternary bên ngoài Show là dư thừa
{data ? <Show when={data}><DataView data={data} /></Show> : <EmptyState />}
```

**Đúng:**

```tsx
// ✅ Sử dụng fallback cho nội dung thay thế
<Show when={data} fallback={<EmptyState />}>
  <DataView data={data} />
</Show>
```

---

### RULE-SHOW-03: Sử dụng render function khi cần giá trị `when` đã được thu hẹp kiểu (narrowed type)

`children` có thể là một function nhận vào giá trị `when` đã được đảm bảo truthy. Sử dụng render function khi cần truy cập giá trị `when` mà không phải ép kiểu hoặc kiểm tra lại null.

**Sai:**

```tsx
// ❌ Phải dùng optional chaining hoặc ép kiểu dù đã kiểm tra trong when
<Show when={selectedUser}>
  <UserProfile name={selectedUser?.name} email={selectedUser?.email} />
</Show>
```

**Đúng:**

```tsx
// ✅ Render function nhận giá trị when đã được đảm bảo truthy
<Show when={selectedUser}>
  {(user) => <UserProfile name={user.name} email={user.email} />}
</Show>
```

---

### RULE-SHOW-04: Không lồng nhiều `Show` khi có thể gộp điều kiện

Khi nhiều điều kiện cần đồng thời thỏa mãn để render, gộp chúng vào một biểu thức `when` duy nhất thay vì lồng nhiều `Show`.

**Sai:**

```tsx
// ❌ Lồng Show gây khó đọc
<Show when={isLoaded}>
  <Show when={hasPermission}>
    <AdminPanel />
  </Show>
</Show>
```

**Đúng:**

```tsx
// ✅ Gộp điều kiện vào một Show duy nhất
<Show when={isLoaded && hasPermission}>
  <AdminPanel />
</Show>
```

---

### RULE-SHOW-05: Không truyền `fallback={null}` -- đây là giá trị mặc định

Khi không cần fallback, không cần truyền `fallback={null}` vì component đã trả về `null` mặc định khi `when` là falsy.

**Sai:**

```tsx
// ❌ Thừa — fallback mặc định đã là null
<Show when={isVisible} fallback={null}>
  <Content />
</Show>
```

**Đúng:**

```tsx
// ✅ Bỏ qua fallback khi không cần nội dung thay thế
<Show when={isVisible}>
  <Content />
</Show>
```

---

## Tham chiếu props

| Prop | Type | Bắt buộc | Mặc định | Mô tả |
|------|------|----------|----------|-------|
| `when` | `T` | Có | - | Điều kiện để render children. Truthy thì render, falsy thì render fallback |
| `children` | `ReactNode \| ((when: T) => ReactNode)` | Có | - | Nội dung hiển thị hoặc render function nhận giá trị `when` |
| `fallback` | `ReactNode` | Không | `null` | Nội dung hiển thị khi `when` là falsy |
