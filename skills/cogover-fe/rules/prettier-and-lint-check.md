---
title: Chạy Prettier và Sửa Lỗi Lint Sau Khi Sửa Code
impact: HIGH
impactDescription: Code không được format và có lỗi lint sẽ gây inconsistency trong codebase, khó review và dễ gây conflict
tags: prettier, lint, eslint, formatting, code-quality
---

## Chạy Prettier và Sửa Lỗi Lint Sau Khi Sửa Code

Sau khi sửa code xong, **bắt buộc** phải thực hiện 2 bước sau:

### Bước 1: Chạy Prettier

Kiểm tra ở thư mục root của dự án có file `.prettierrc` không. Nếu có, chạy prettier với tất cả các file vừa sửa:

```bash
# Kiểm tra file .prettierrc tồn tại
ls .prettierrc*

# Chạy prettier cho các file vừa sửa
npx prettier --write <file1> <file2> ...
```

### Bước 2: Kiểm tra và sửa lỗi lint

Kiểm tra lỗi lint trong tất cả các file vừa sửa và sửa hết các lỗi:

```bash
# Kiểm tra lỗi lint
npx eslint <file1> <file2> ...

# Hoặc sửa tự động nếu có thể
npx eslint --fix <file1> <file2> ...
```

### Quy trình:

1. Sửa code xong
2. Kiểm tra `.prettierrc` ở root → nếu có thì chạy `prettier --write` với các file vừa sửa
3. Chạy lint check cho các file vừa sửa
4. Sửa tất cả lỗi lint cho đến khi không còn lỗi

### Lưu ý:

- **Luôn thực hiện** sau mỗi lần sửa code, không cần hỏi lại
- Chạy prettier **trước** khi kiểm tra lint để tránh lỗi formatting bị báo là lint error
- Sửa **tất cả** lỗi lint, không bỏ sót
