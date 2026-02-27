---
title: Không Tự Động Tạo File Markdown
impact: HIGH
impactDescription: Tự động tạo file markdown gây ra file thừa trong codebase, khó quản lý và dễ bị outdated
tags: markdown, documentation, workflow, best-practice
---

## Không Tự Động Tạo File Markdown

Không bao giờ tự động tạo các file markdown (`.md`) như README, documentation, changelog, hoặc bất kỳ file `.md` nào khác trừ khi user yêu cầu rõ ràng.

### Sai:

```
// ❌ Tự tạo README.md khi thêm feature mới
// ❌ Tự tạo CHANGELOG.md khi sửa code
// ❌ Tự tạo docs/*.md để giải thích code
// ❌ Tự tạo file .md bất kỳ mà không được yêu cầu
```

### Đúng:

```
// ✅ Chỉ tạo file markdown khi user yêu cầu rõ ràng
// ✅ Ưu tiên viết comment trong code thay vì tạo file markdown riêng
// ✅ Nếu cần giải thích, trả lời trực tiếp trong conversation
```
