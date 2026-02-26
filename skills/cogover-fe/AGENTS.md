# Hướng dẫn tạo Commit

Khi được yêu cầu "tạo commit" (create commit), thực hiện các bước dưới đây mà không cần hỏi xác nhận.

---

## Bước 1: Kiểm tra tên nhánh hiện tại

Chạy lệnh sau để lấy tên nhánh hiện tại:

```bash
git branch --show-current
```

---

## Bước 2: Phát hiện mã task từ tên nhánh

- Định dạng mã task: `SPT-<số>` hoặc `SPT_<số>`
- Ví dụ: `feature/SPT-911-fix-bug`, `bugfix/SPT_123_update_ui`

**Nếu tìm thấy mã task** → dùng `[SPT-<số>]` làm prefix.

**Nếu không tìm thấy mã task** → dùng `[<tên-nhánh-đầy-đủ>]` làm prefix.

---

## Bước 3: Xem lại các thay đổi đã stage và chưa stage

Chạy các lệnh sau song song để hiểu những gì sẽ được commit:

```bash
git status
git diff
git diff --staged
```

---

## Bước 4: Soạn commit message

### Định dạng khi có mã task:
```
[<mã-task>] <mô tả bằng tiếng Anh>
```

**Ví dụ:**
- Nhánh: `feature/SPT-911-fix-bug-prod`
- Commit: `[SPT-911] Add allowHideFunction and allowShowFunction to Popper component`

### Định dạng khi không có mã task:
```
[<tên-nhánh>] <mô tả bằng tiếng Anh>
```

**Ví dụ:**
- Nhánh: `hotfix/update-dependencies`
- Commit: `[hotfix/update-dependencies] Update React to version 18.2.0`

---

## Bước 5: Quy tắc viết mô tả commit

- Viết bằng **tiếng Anh**
- **Ngắn gọn và đầy đủ ý** — tối đa 72 ký tự cho dòng đầu tiên
- Sử dụng **động từ ở dạng mệnh lệnh** (imperative mood): `Add`, `Fix`, `Update`, `Remove`, `Refactor`, `Rename`, `Move`, `Extract`, ...
- Mô tả **điều gì đã thay đổi**, không phải tại sao thay đổi

### Commit message tốt:
```
[SPT-911] Add allowHideFunction prop to Popper component
[SPT-456] Fix modal not closing when clicking outside
[SPT-789] Update form validation logic
[SPT-123] Remove deprecated API calls
[SPT-321] Refactor user authentication flow
```

### Commit message không tốt:
```
❌ fixed bug
❌ update
❌ WIP
❌ Sửa lỗi form
❌ [SPT-911] Đã thêm chức năng mới cho component
```

---

## Bước 6: Stage và commit

Stage các file cụ thể (ưu tiên hơn `git add .`):

```bash
git add <file1> <file2> ...
```

Sau đó commit sử dụng HEREDOC để giữ nguyên formatting:

```bash
git commit -m "$(cat <<'EOF'
[SPT-911] Add allowHideFunction and allowShowFunction to Popper component
EOF
)"
```

---

## Bước 7: Xác nhận commit

```bash
git status
git log --oneline -5
```

---

## Lưu ý quan trọng

- **KHÔNG push** lên remote trừ khi được yêu cầu rõ ràng.
- **KHÔNG sử dụng** `--no-verify` hoặc bỏ qua hooks trừ khi được yêu cầu rõ ràng.
- **KHÔNG amend** commit trước đó trừ khi được yêu cầu rõ ràng. Nếu pre-commit hook thất bại, hãy sửa lỗi và tạo một **commit mới**.
- **KHÔNG sử dụng** `git add -A` hoặc `git add .` — stage từng file cụ thể để tránh vô tình thêm các file nhạy cảm (`.env`, credentials, large binaries).
- **KHÔNG tạo empty commit** nếu không có thay đổi nào để stage.


---

# Hướng dẫn Dịch i18n cho UI

## Bước 1: Hỏi lại thông tin cần thiết

Khi được yêu cầu "dịch các text tiếng Việt", **LUÔN hỏi lại** 2 thông tin sau trước khi làm:

1. **Namespace** là gì? (ví dụ: `CHAT`, `CONTACT`, `COMMON`, ...)
2. **Prefix** là gì? (ví dụ: `contactDetail`, `chatMessage`, ...)

---

## Bước 2: Xác định loại dự án để import đúng

Kiểm tra loại dự án để dùng import phù hợp:

### Dự án router
```tsx
import { useTranslation, Trans } from 'react-i18next';
```

### Dự án ui-kit
Đọc file `src/hooks/i18n.ts` để xem cách định nghĩa hook, sau đó tạo hook tương tự theo những phần đã làm trong file đó.

### Các dự án còn lại (mặc định)
```tsx
import { useTranslation } from 'src/languages/global';
import Trans from 'src/languages/global/Trans';
```

---

## Bước 3: Lấy I18nNS từ namespace

Vào file `src/languages/i18n.ts` để lấy enum `I18nNS` tương ứng với namespace được yêu cầu.

Ví dụ: namespace = `CHAT` → tìm `I18nNS.CHAT` trong file đó.

Sau đó tạo function `t` trong component:
```tsx
const { t: tChat } = useTranslation(I18nNS.CHAT);
```

> Tên biến `t` đặt theo namespace: `tChat`, `tContact`, `tCommon`, ...

---

## Bước 4: Quy tắc dịch text

### 4.1 Chuỗi thông thường (plain string)

Dịch text tiếng Việt sang tiếng Anh bình thường.

**Ví dụ:**
- `"Xác nhận"` → `"Confirm"`
- `"Hủy bỏ"` → `"Cancel"`
- `"Đang tải..."` → `"Loading..."`

### 4.2 Chuỗi có placeholder

Dịch text tiếng Việt sang tiếng Anh, giữ nguyên placeholder.

**Ví dụ:**
- `"Xin chào {{name}}"` → `"Hello {{name}}"`

> Lưu ý: placeholder trong i18n dùng dấu `{{}}` kép.

### 4.3 Template HTML

Dịch text tiếng Việt sang tiếng Anh, chuyển các tag HTML thành số thứ tự `<1>`, `<2>`, ...

**Ví dụ:**
- Tiếng Việt: `"Đây là chữ <strong>bôi đậm</strong>: <em>{{name}}</em>"`
- Tiếng Anh (i18n key value): `"This is <1>bold</1> text: <2>{{name}}</2>"`

---

## Bước 5: Tạo bảng so sánh và in ra terminal

Sau khi dịch, tạo bảng markdown so sánh với các cột theo thứ tự:

| key | Tiếng Việt | Tiếng Anh |
|-----|-----------|-----------|

- Cột **key**: camelCase của text tiếng Anh, **không** bao gồm prefix
- Cột **Tiếng Việt**: text gốc
- Cột **Tiếng Anh**: text đã dịch (giá trị i18n)

**Ví dụ:**

| key | Tiếng Việt | Tiếng Anh |
|-----|-----------|-----------|
| `confirm` | Xác nhận | Confirm |
| `cancel` | Hủy bỏ | Cancel |
| `greeting` | Xin chào {{name}} | Hello {{name}} |

> **In bảng này ra terminal** để người dùng có thể copy vào file ngôn ngữ.

---

## Bước 6: Sử dụng trong code

### 6.1 Chuỗi thông thường
```tsx
{tChat("contactDetail.confirm")}
```

### 6.2 Chuỗi có placeholder
```tsx
{tChat("contactDetail.greeting", { name: userName })}
```

### 6.3 Template HTML
```tsx
<Trans
  ns={I18nNS.CHAT}
  i18nKey="contactDetail.boldText"
  components={{ 1: <strong />, 2: <em /> }}
  values={{ name: "name value" }}
/>
```

> Số trong `components` tương ứng với số thứ tự tag trong key: `<1>`, `<2>`, ...

---

## Ví dụ đầy đủ

### Input
- Namespace: `CHAT`
- Prefix: `contactDetail`
- Text cần dịch:
  - `"Xác nhận"`
  - `"Xin chào {{name}}"`
  - `"Đây là chữ <strong>bôi đậm</strong>: <em>{{name}}</em>"`

### Output - Bảng dịch (in ra terminal)

| key | Tiếng Việt | Tiếng Anh |
|-----|-----------|-----------|
| `confirm` | Xác nhận | Confirm |
| `greeting` | Xin chào {{name}} | Hello {{name}} |
| `boldText` | Đây là chữ <strong>bôi đậm</strong>: <em>{{name}}</em> | This is <1>bold</1> text: <2>{{name}}</2> |

### Output - Code

```tsx
import { useTranslation } from 'src/languages/global';
import Trans from 'src/languages/global/Trans';
import { I18nNS } from 'src/languages/i18n';

const { t: tChat } = useTranslation(I18nNS.CHAT);

// Chuỗi thông thường
{tChat("contactDetail.confirm")}

// Chuỗi có placeholder
{tChat("contactDetail.greeting", { name: userName })}

// Template HTML
<Trans
  ns={I18nNS.CHAT}
  i18nKey="contactDetail.boldText"
  components={{ 1: <strong />, 2: <em /> }}
  values={{ name: userName }}
/>
```

---

## Lưu ý quan trọng

- **Không tạo file ngôn ngữ** (file JSON/translations). Chỉ cung cấp bảng dịch để người dùng tự thêm.
- **Không** đưa prefix vào cột `key` của bảng.
- Placeholder trong i18n dùng `{{name}}` (dấu ngoặc kép), không phải `{name}`.
- Số thứ tự trong `components` của `<Trans>` bắt đầu từ `1`.
