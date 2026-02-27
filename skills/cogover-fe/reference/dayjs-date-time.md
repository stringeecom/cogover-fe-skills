## Luôn sử dụng dayjs khi làm việc với date/time

Dayjs là thư viện duy nhất trong dự án cho date/time. Tất cả components DatePicker, DateRangePicker, TimePicker, Calendar dùng `Dayjs` type.

```ts
import dayjs from 'dayjs';

// Thời gian hiện tại
const now = dayjs();                          // ✅  (không dùng new Date())

// Parse
const date = dayjs('2024-01-15');             // từ ISO string
const dateVN = dayjs('15/01/2024', 'DD/MM/YYYY'); // custom format
const date2 = dayjs(1705276800000);           // từ timestamp
const date3 = dayjs(nativeDateObj);           // từ Date object

// Format
dayjs().format('DD/MM/YYYY')           // 15/01/2024
dayjs().format('DD/MM/YYYY HH:mm')     // 15/01/2024 14:30
dayjs().format('HH:mm:ss')             // 14:30:00

// So sánh
dayjs(d1).isBefore(dayjs(d2));
dayjs(d1).isAfter(dayjs(d2));
dayjs(d1).isSame(dayjs(d2), 'day');

// Tính toán
dayjs().add(7, 'day');
dayjs().subtract(1, 'month');
dayjs(d2).diff(d1, 'day');             // khoảng cách

// Đầu/cuối ngày
dayjs().startOf('day');                // 00:00:00.000
dayjs().endOf('day');                  // 23:59:59.999

// Timestamp (gửi API)
const ts = dayjs().valueOf();
const startTs = dayjs(startDate).startOf('day').valueOf();
const endTs = dayjs(endDate).endOf('day').valueOf();

// Validation
dayjs(value).isValid();
dayjs(value, 'DD/MM/YYYY', true).isValid(); // strict mode

// Lấy đơn vị
dayjs().year();
dayjs().month() + 1;  // 0-based
```

| Thao tác | DON'T | DO |
|----------|-------|-----|
| Hiện tại | `new Date()` / `Date.now()` | `dayjs()` |
| Parse | `new Date('2024-01-15')` | `dayjs('2024-01-15')` |
| Format | `date.toLocaleDateString()` | `dayjs().format('DD/MM/YYYY')` |
| Timestamp | `date.getTime()` | `dayjs().valueOf()` |
| So sánh | `d1.getTime() < d2.getTime()` | `dayjs(d1).isBefore(d2)` |
| Cộng ngày | tự tính ms | `dayjs().add(7, 'day')` |
| Khoảng cách | tự tính ms rồi chia | `dayjs(d2).diff(d1, 'day')` |
| Đầu ngày | `setHours(0,0,0,0)` | `dayjs().startOf('day')` |
| Validate | `!isNaN(new Date(v).getTime())` | `dayjs(v).isValid()` |
