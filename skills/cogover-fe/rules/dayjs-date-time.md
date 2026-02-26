---
title: Luôn sử dụng dayjs khi làm việc với date/time
impact: HIGH
impactDescription: Sử dụng native Date, moment.js, hoặc tự tính toán date/time dẫn đến code không nhất quán, lỗi timezone, format sai, và không tương thích với các component DatePicker/DateRangePicker/TimePicker của ui-kit
tags: dayjs, date, time, datetime, timestamp, format, parse, timezone
---

## Luôn sử dụng dayjs khi làm việc với date/time

Trong dự án, **dayjs** là thư viện duy nhất được sử dụng cho mọi thao tác liên quan đến ngày giờ. Tất cả component DatePicker, DateRangePicker, TimePicker, Calendar của ui-kit đều sử dụng `Dayjs` type.

---

### RULE-DAYJS-01: Luôn import dayjs từ `dayjs`, không sử dụng native Date hoặc moment.js

**Sai:**

```ts
// ❌ Native Date
const now = new Date();
const yesterday = new Date(Date.now() - 86400000);
const formatted = new Date().toLocaleDateString('vi-VN');

// ❌ moment.js
import moment from 'moment';
const date = moment('2024-01-15');

// ❌ Tự tính toán với timestamp
const startOfDay = Math.floor(Date.now() / 86400000) * 86400000;
```

**Đúng:**

```ts
// ✅ dayjs
import dayjs from 'dayjs';

const now = dayjs();
const yesterday = dayjs().subtract(1, 'day');
const formatted = dayjs().format('DD/MM/YYYY');
```

---

### RULE-DAYJS-02: Sử dụng dayjs để parse date từ mọi nguồn (string, timestamp, Date object)

**Sai:**

```ts
// ❌ Tự parse string
const parts = '15/01/2024'.split('/');
const date = new Date(Number(parts[2]), Number(parts[1]) - 1, Number(parts[0]));

// ❌ Truyền timestamp trực tiếp vào component
<DatePicker value={1705276800000} />
```

**Đúng:**

```ts
// ✅ Parse từ string
const date = dayjs('2024-01-15');
const dateVN = dayjs('15/01/2024', 'DD/MM/YYYY');

// ✅ Parse từ timestamp
const date = dayjs(1705276800000);

// ✅ Parse từ Date object (khi nhận từ bên ngoài)
const date = dayjs(nativeDateFromExternalLib);

// ✅ Truyền Dayjs vào component
<DatePicker value={dayjs(timestamp)} />
```

---

### RULE-DAYJS-03: Sử dụng các method của dayjs để format, không tự build string

**Sai:**

```ts
// ❌ Tự build string format
const date = new Date();
const formatted = `${date.getDate()}/${date.getMonth() + 1}/${date.getFullYear()}`;

// ❌ Template literal với getHours/getMinutes
const time = `${date.getHours()}:${String(date.getMinutes()).padStart(2, '0')}`;
```

**Đúng:**

```ts
// ✅ dayjs format
const formatted = dayjs().format('DD/MM/YYYY');
const time = dayjs().format('HH:mm');
const full = dayjs().format('DD/MM/YYYY HH:mm:ss');

// ✅ Format phổ biến trong dự án
dayjs(timestamp).format('DD/MM/YYYY')           // 15/01/2024
dayjs(timestamp).format('DD/MM/YYYY HH:mm')     // 15/01/2024 14:30
dayjs(timestamp).format('HH:mm:ss')             // 14:30:00
dayjs(timestamp).format('DD/MM/YYYY HH:mm:ss')  // 15/01/2024 14:30:00
```

---

### RULE-DAYJS-04: Sử dụng các method của dayjs để so sánh và tính toán, không tự tính

**Sai:**

```ts
// ❌ Tự tính khoảng cách
const diffMs = date2.getTime() - date1.getTime();
const diffDays = Math.floor(diffMs / (1000 * 60 * 60 * 24));

// ❌ Tự so sánh
const isBefore = date1.getTime() < date2.getTime();

// ❌ Tự tính ngày bắt đầu/kết thúc
const startOfDay = new Date(date);
startOfDay.setHours(0, 0, 0, 0);
```

**Đúng:**

```ts
// ✅ So sánh
dayjs(date1).isBefore(dayjs(date2));
dayjs(date1).isAfter(dayjs(date2));
dayjs(date1).isSame(dayjs(date2), 'day');

// ✅ Tính khoảng cách
dayjs(date2).diff(dayjs(date1), 'day');
dayjs(date2).diff(dayjs(date1), 'hour');

// ✅ Thao tác (add, subtract)
dayjs().add(7, 'day');
dayjs().subtract(1, 'month');

// ✅ Đầu/cuối ngày
dayjs().startOf('day');    // 00:00:00.000
dayjs().endOf('day');      // 23:59:59.999

// ✅ Đầu/cuối tháng
dayjs().startOf('month');
dayjs().endOf('month');
```

---

### RULE-DAYJS-05: Sử dụng dayjs khi chuyển đổi giữa Dayjs và timestamp

Khi gửi API thường cần timestamp (number), khi nhận API thường nhận timestamp hoặc ISO string. Luôn dùng dayjs để chuyển đổi.

**Sai:**

```ts
// ❌ Dùng Date.now() hoặc getTime()
const timestamp = new Date().getTime();
const timestamp = Date.now();
const fromString = new Date('2024-01-15').getTime();
```

**Đúng:**

```ts
// ✅ Dayjs → timestamp (gửi API)
const timestamp = dayjs().valueOf();
const timestamp = dayjs('2024-01-15').valueOf();

// ✅ Timestamp → Dayjs (nhận API)
const date = dayjs(response.created);
const date = dayjs(response.timestamp);

// ✅ Normalize cho date range gửi API
const startTimestamp = dayjs(startDate).startOf('day').valueOf();   // 00:00:00
const endTimestamp = dayjs(endDate).endOf('day').valueOf();         // 23:59:59
```

---

### RULE-DAYJS-06: Sử dụng dayjs để kiểm tra tính hợp lệ của date

**Sai:**

```ts
// ❌ Tự kiểm tra
const isValid = !isNaN(new Date(value).getTime());

// ❌ Regex
const isValid = /^\d{2}\/\d{2}\/\d{4}$/.test(value);
```

**Đúng:**

```ts
// ✅ dayjs isValid
const isValid = dayjs(value).isValid();
const isValid = dayjs(value, 'DD/MM/YYYY', true).isValid(); // strict mode
```

---

### RULE-DAYJS-07: Không sử dụng `new Date()` để lấy thời gian hiện tại

**Sai:**

```ts
// ❌
const now = new Date();
const timestamp = Date.now();
const year = new Date().getFullYear();
const month = new Date().getMonth() + 1;
```

**Đúng:**

```ts
// ✅
const now = dayjs();
const timestamp = dayjs().valueOf();
const year = dayjs().year();
const month = dayjs().month() + 1; // dayjs month cũng 0-based
```

---

## Tham chiếu nhanh

| Thao tác | Native Date (❌) | dayjs (✅) |
|----------|-----------------|-----------|
| Thời gian hiện tại | `new Date()` | `dayjs()` |
| Parse string | `new Date('2024-01-15')` | `dayjs('2024-01-15')` |
| Parse timestamp | `new Date(1705276800000)` | `dayjs(1705276800000)` |
| Format | `date.toLocaleDateString()` | `dayjs().format('DD/MM/YYYY')` |
| Timestamp | `date.getTime()` | `dayjs().valueOf()` |
| So sánh | `d1.getTime() < d2.getTime()` | `dayjs(d1).isBefore(d2)` |
| Cộng ngày | Tự tính ms | `dayjs().add(7, 'day')` |
| Trừ ngày | Tự tính ms | `dayjs().subtract(1, 'month')` |
| Đầu ngày | `date.setHours(0,0,0,0)` | `dayjs().startOf('day')` |
| Cuối ngày | `date.setHours(23,59,59)` | `dayjs().endOf('day')` |
| Khoảng cách | Tự tính ms rồi chia | `dayjs(d2).diff(d1, 'day')` |
| Kiểm tra hợp lệ | `!isNaN(new Date(v).getTime())` | `dayjs(v).isValid()` |
| Lấy năm | `date.getFullYear()` | `dayjs().year()` |
| Lấy tháng | `date.getMonth() + 1` | `dayjs().month() + 1` |
