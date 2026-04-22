---
title: Hướng dẫn Dịch i18n cho UI
tags: i18n, translation, react-i18next
---

# Hướng dẫn Dịch i18n cho UI

## Bước 0 (BẮT BUỘC): Xác định dự án static

**TRƯỚC KHI LÀM BẤT CỨ VIỆC GÌ KHÁC**, phải tìm được dự án static và biết chính xác nó nằm ở đâu.

- Dự án static **thường nằm ngang cấp** với dự án hiện tại (cùng thư mục cha).
- File ngôn ngữ nằm ở: `build_docker/static/locales/<locale>/<namespace>.json`
  - Ví dụ: `build_docker/static/locales/vi-VN/common.json`, `build_docker/static/locales/en-US/chat.json`
- Nếu **KHÔNG tìm thấy** dự án static → **DỪNG LẠI** và yêu cầu user cung cấp đường dẫn, KHÔNG được tự đoán.

### Kiểm tra git branch của dự án static

Sau khi tìm được dự án static, phải kiểm tra branch hiện tại của nó có **trùng khớp** với branch của dự án hiện tại không.

- Nếu **khác branch** → **CẢNH BÁO cho user** biết ngay, yêu cầu user checkout/tạo nhánh hợp lệ bên dự án static trước khi tiếp tục.
- Không tự ý checkout/tạo nhánh bên dự án static.

---

## Bước 1: Hỏi lại thông tin cần thiết

Khi được yêu cầu "dịch các text tiếng Việt", **LUÔN hỏi lại** 2 thông tin sau trước khi làm:

1. **Namespace** là gì? (ví dụ: `CHAT`, `CONTACT`, `COMMON`, ...)
2. **Prefix** là gì? (ví dụ: `contactDetail`, `chatMessage`, ...)

### Quy tắc prefix đặc biệt (CRITICAL — chỉ áp dụng cho key MỚI)

> ⚠️ **NHẤN MẠNH**: Với **mọi key mới**, BẮT BUỘC phải tách theo loại nội dung:
>
> - Text thuộc **message validate** (lỗi form, lỗi nhập liệu, required, min/max, định dạng sai, ...) → **luôn** dùng prefix `schema`.
> - Text thuộc **thông báo** (toast, alert, message báo thành công/thất bại sau thao tác, ...) → **luôn** dùng prefix `notification`.
>
> Các key CŨ không tuân theo rule này thì **KHÔNG cần sửa**, giữ nguyên hiện trạng. Rule này **chỉ** áp dụng khi tạo key **mới**.

---

## Bước 2 (BẮT BUỘC — KHÔNG BAO GIỜ ĐƯỢC BỎ QUA): Tìm key tương đồng trong namespace COMMON

> ⚠️ **CỰC KỲ QUAN TRỌNG**: Bước này **KHÔNG BAO GIỜ** được bỏ qua dưới bất kỳ lý do nào. Mọi lần dịch đều BẮT BUỘC phải mở và đọc file `common.json` trước khi tạo key mới.

**TRƯỚC KHI tạo key mới**, luôn mở file `build_docker/static/locales/vi-VN/common.json` của dự án static và tìm xem đã có key nào có bản dịch **tương đồng** (về nghĩa) với text đang cần dịch hay chưa.

- Nếu tìm thấy → **DÙNG LUÔN** key đó, không tạo mới.
- Các key dùng chung phổ biến: `save`, `cancel`, `confirm`, `delete`, `edit`, `close`, `submit`, `search`, `back`, ...
- **Nếu KHÔNG tìm thấy file `common.json`** (không tìm thấy dự án static, sai đường dẫn, file không tồn tại, ...) → **DỪNG LẠI NGAY và HỎI LẠI USER**, tuyệt đối **KHÔNG** được tự đoán, tự bỏ qua, hay tự tạo key mới mà chưa tra cứu.

---

## Bước 3: Xác định loại dự án để import đúng

### Dự án router
```tsx
import { useTranslation, Trans } from 'react-i18next';
```

### Dự án ui-kit
Đọc file `src/hooks/i18n.ts` để xem cách định nghĩa hook, sau đó tạo hook tương tự.

### Các dự án còn lại (mặc định)
```tsx
import { useTranslation } from 'src/languages/global';
import Trans from 'src/languages/global/Trans';
```

---

## Bước 4: Lấy I18nNS từ namespace

Vào file `src/languages/i18n.ts` để lấy enum `I18nNS` tương ứng.

```tsx
const { t: tChat } = useTranslation(I18nNS.CHAT);
```

> Tên biến `t` đặt theo namespace: `tChat`, `tContact`, `tCommon`, ...

---

## Bước 5: Quy tắc dịch text

### 5.1 Chuỗi thông thường
- `"Xác nhận"` → `"Confirm"`

### 5.2 Chuỗi có placeholder
- `"Xin chào {{name}}"` → `"Hello {{name}}"`
> Placeholder dùng `{{}}` kép.

### 5.3 Template HTML
- Tiếng Việt: `"Đây là chữ <strong>bôi đậm</strong>: <em>{{name}}</em>"`
- Tiếng Anh: `"This is <1>bold</1> text: <2>{{name}}</2>"`

---

## Bước 6: Tạo bảng so sánh và in ra terminal

| key | Tiếng Việt | Tiếng Anh |
|-----|-----------|-----------|
| `confirm` | Xác nhận | Confirm |
| `greeting` | Xin chào {{name}} | Hello {{name}} |

- Cột **key**: camelCase của text tiếng Anh, **không** bao gồm prefix

### Quy tắc không trùng key (CRITICAL)

**KHÔNG được tạo key có tên trùng với 1 key khác trong CÙNG 1 namespace + CÙNG prefix.**

- Ví dụ SAI: trong namespace `CHAT` đã có key `channelSettings.facebook` → **KHÔNG** được tạo thêm `channelSettings.facebook` nữa.
- Ví dụ ĐÚNG: trong namespace `CHAT` đã có key `channelSettings.facebook` → VẪN được tạo key `integration.facebook` (khác prefix).

Trước khi thêm key mới vào file JSON, phải kiểm tra full path `<prefix>.<key>` chưa tồn tại.

### Quy tắc độ sâu key (CRITICAL)

**KHÔNG BAO GIỜ được tạo key ở cấp thứ 3.** Key **luôn** có dạng `<prefix>.<name>`, tối đa 2 cấp.

- Ví dụ ĐÚNG: `schema.emailRequired`, `notification.saveSuccess`, `contactDetail.confirm`.
- Ví dụ SAI: `schema.email.required`, `notification.save.success`, `contactDetail.form.confirm` — đây là dạng `<prefix>.<name_1>.<name_2>`, **tuyệt đối không** được dùng.

Nếu cần biểu diễn thêm ngữ cảnh, **gộp vào cùng `<name>`** bằng camelCase thay vì tách thành cấp con.

---

## Bước 7: Sử dụng trong code

### 7.1 Chuỗi thông thường
```tsx
{tChat("contactDetail.confirm")}
```

### 7.2 Chuỗi có placeholder
```tsx
{tChat("contactDetail.greeting", { name: userName })}
```

### 7.3 Template HTML
```tsx
<Trans
  ns={I18nNS.CHAT}
  i18nKey="contactDetail.boldText"
  components={{ 1: <strong />, 2: <em /> }}
  values={{ name: userName }}
/>
```

> Số trong `components` tương ứng với số thứ tự tag: `<1>`, `<2>`, ...

---

## Bước 8 (BẮT BUỘC): Update file ngôn ngữ của dự án static

Sau khi dịch xong và không còn thắc mắc gì nữa, **tự động update** luôn các key mới vào file ngôn ngữ của dự án static — **KHÔNG** bắt user tự update.

- Update vào tất cả các locale tương ứng: `build_docker/static/locales/vi-VN/<namespace>.json` và `build_docker/static/locales/en-US/<namespace>.json` (và các locale khác nếu có).
- Giữ nguyên format JSON, thứ tự key hợp lý theo prefix.
- Nếu còn thắc mắc về nghĩa/placeholder/HTML tag → hỏi user trước, CHƯA update.

### Ngoại lệ: Dự án live-chat

> Nếu đang sửa i18n cho dự án **live-chat** → update thẳng vào `src/languages/locales/<locale>/<namespace>.json` **của chính dự án live-chat**, **KHÔNG** update sang dự án static.

---

## Lưu ý quan trọng

- **Bước 0 và Bước 2 là bắt buộc** — luôn tìm dự án static + tra key COMMON trước khi dịch.
- **Bước 2 KHÔNG BAO GIỜ được bỏ qua.** Nếu không tìm thấy `common.json` → DỪNG và HỎI LẠI USER, không tự đoán.
- **Dự án live-chat**: update i18n vào `src/languages/locales` của chính dự án live-chat, **không** dùng dự án static.
- **Không** đưa prefix vào cột `key` của bảng.
- Placeholder dùng `{{name}}` (ngoặc kép), không phải `{name}`.
- Số thứ tự trong `components` của `<Trans>` bắt đầu từ `1`.
- Không trùng key trong cùng namespace + cùng prefix.
- **Key mới** thuộc message validate → prefix `schema`; thuộc thông báo → prefix `notification` (key cũ giữ nguyên).
- Key **tối đa 2 cấp** (`<prefix>.<name>`), tuyệt đối không tạo cấp thứ 3.
