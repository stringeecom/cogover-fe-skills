---
title: Cách sử dụng component DateTimeWithTooltip trong ui-kit
impact: MEDIUM
impactDescription: Sử dụng sai dẫn đến hiển thị ngày giờ không nhất quán, mất tooltip timezone, hoặc duplicate logic format ngày giờ
tags: datetime, tooltip, timezone, time, format, meridiem, ui-kit
---

## Cách sử dụng component DateTimeWithTooltip trong ui-kit

Đường dẫn component: `ui-kit/src/components/DateTimeWithTooltip`
Xây dựng trên: `Tooltip`, `useFormatDateTime`, `useSelectDisplayDateFormat`, `useSelectDisplayTimeFormat`, `useDisplayLanguage`

Props: `time` (number | string | undefined), cộng tất cả `React.HTMLAttributes<HTMLSpanElement>` (className, style, onClick, v.v.).

Hành vi chính:
- Hiển thị ngày giờ đã format theo cài đặt người dùng (date format, time format, ngôn ngữ).
- Hover hiển thị tooltip chứa ngày giờ đầy đủ (bao gồm thứ, giây) và thông tin timezone (GMT offset).
- Tự động chuyển đổi AM/PM sang SA/CH khi ngôn ngữ là `vi-VN`.
- Trả về `null` khi `time` là falsy (undefined, null, 0, "").

---

### RULE-DTWT-01: `time` phải là timestamp (number) hoặc date string — không truyền Dayjs, Date object

`time` nhận `number` (Unix timestamp milliseconds) hoặc `string` (ISO date string). Không truyền `Dayjs` instance hay native `Date` object.

**Sai:**

```tsx
// ❌ Dayjs instance
<DateTimeWithTooltip time={dayjs()} />

// ❌ Native Date object
<DateTimeWithTooltip time={new Date()} />
```

**Đúng:**

```tsx
// ✅ Unix timestamp (milliseconds)
<DateTimeWithTooltip time={1705276800000} />

// ✅ ISO date string
<DateTimeWithTooltip time="2024-01-15T08:00:00Z" />

// ✅ Timestamp từ API response
<DateTimeWithTooltip time={ticket.createdAt} />
```

---

### RULE-DTWT-02: Không tự format ngày giờ + wrap Tooltip thủ công — sử dụng DateTimeWithTooltip

Component đã tích hợp sẵn logic format theo cài đặt người dùng (date format, time format, ngôn ngữ) và tooltip hiển thị ngày giờ đầy đủ kèm timezone. Không tự kết hợp `Tooltip` + `dayjs().format()` thủ công.

**Sai:**

```tsx
// ❌ Tự format và wrap tooltip — mất đồng bộ với cài đặt user
<Tooltip content={dayjs(time).format("dddd, DD/MM/YYYY, HH:mm:ss")}>
  <span>{dayjs(time).format("DD/MM/YYYY HH:mm")}</span>
</Tooltip>

// ❌ Tự format không có tooltip
<span>{dayjs(ticket.createdAt).format("DD/MM/YYYY HH:mm")}</span>
```

**Đúng:**

```tsx
// ✅ Component tự xử lý format và tooltip
<DateTimeWithTooltip time={ticket.createdAt} />
```

---

### RULE-DTWT-03: Không cần kiểm tra `time` trước khi render — component tự trả về `null`

Khi `time` là falsy (undefined, null, 0, ""), component tự trả về `null`. Không cần conditional render bên ngoài.

**Sai:**

```tsx
// ❌ Kiểm tra thừa
{ticket.createdAt ? <DateTimeWithTooltip time={ticket.createdAt} /> : null}

// ❌ Kiểm tra thừa với &&
{ticket.createdAt && <DateTimeWithTooltip time={ticket.createdAt} />}
```

**Đúng:**

```tsx
// ✅ Component tự xử lý falsy value
<DateTimeWithTooltip time={ticket.createdAt} />
```

---

### RULE-DTWT-04: Sử dụng spread HTMLAttributes để tuỳ chỉnh — không wrap thêm span/div

Component render một `<span>` element và chấp nhận tất cả `HTMLAttributes<HTMLSpanElement>`. Truyền `className`, `style`, `onClick`, v.v. trực tiếp vào component thay vì wrap thêm element bên ngoài.

**Sai:**

```tsx
// ❌ Wrap thêm span không cần thiết
<span className={cx("text-neutral-400 prose-caption1")} onClick={handleClick}>
  <DateTimeWithTooltip time={ticket.createdAt} />
</span>
```

**Đúng:**

```tsx
// ✅ Truyền trực tiếp vào component
<DateTimeWithTooltip
  time={ticket.createdAt}
  className={cx("text-neutral-400 prose-caption1")}
  onClick={handleClick}
/>
```

---

### RULE-DTWT-05: Không tự xử lý logic AM/PM sang SA/CH — component tự chuyển đổi theo ngôn ngữ

Component sử dụng `useDisplayLanguage` để lấy ngôn ngữ người dùng và tự động chuyển đổi AM/PM sang SA/CH khi ngôn ngữ là `vi-VN`. Không tự xử lý logic này bên ngoài.

**Sai:**

```tsx
// ❌ Tự chuyển đổi meridiem
const time = dayjs(ticket.createdAt).format("DD/MM/YYYY hh:mm A");
const displayTime = userLang === "vi-VN"
  ? time.replace("AM", "SA").replace("PM", "CH")
  : time;
<Tooltip content={fullDate}>
  <span>{displayTime}</span>
</Tooltip>
```

**Đúng:**

```tsx
// ✅ Component tự xử lý chuyển đổi meridiem theo ngôn ngữ
<DateTimeWithTooltip time={ticket.createdAt} />
```

---

### RULE-DTWT-06: Không wrap DateTimeWithTooltip với Tooltip bên ngoài — tooltip đã tích hợp sẵn

Component đã sử dụng `Tooltip` bên trong để hiển thị ngày giờ đầy đủ và timezone khi hover. Không thêm `Tooltip` bên ngoài vì sẽ gây xung đột tooltip kép.

**Sai:**

```tsx
// ❌ Tooltip kép — xung đột với tooltip có sẵn
<Tooltip content="Ngày tạo">
  <DateTimeWithTooltip time={ticket.createdAt} />
</Tooltip>
```

**Đúng:**

```tsx
// ✅ Tooltip được xử lý nội bộ, hiển thị đầy đủ ngày giờ + timezone
<DateTimeWithTooltip time={ticket.createdAt} />
```
