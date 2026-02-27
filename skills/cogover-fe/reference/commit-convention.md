---
title: Quy ước tạo Commit cho Cogover
tags: commit, git, convention
---

# Hướng dẫn tạo Commit

Khi được yêu cầu "tạo commit" (create commit), thực hiện các bước dưới đây mà không cần hỏi xác nhận.

---

## Bước 1: Kiểm tra tên nhánh hiện tại

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
```

### Commit message không tốt:
```
❌ fixed bug
❌ update
❌ WIP
❌ Sửa lỗi form
```

---

## Bước 6: Stage và commit

Stage các file cụ thể (ưu tiên hơn `git add .`):

```bash
git add <file1> <file2> ...
```

Sau đó commit sử dụng HEREDOC:

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
- **KHÔNG sử dụng** `git add -A` hoặc `git add .` — stage từng file cụ thể.
- **KHÔNG tạo empty commit** nếu không có thay đổi nào để stage.
