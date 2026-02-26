---
title: Không tự động tạo commit
impact: HIGH
impactDescription: Tự động commit khiến user không kịp review code, có thể commit code chưa hoàn chỉnh hoặc sai yêu cầu
tags: workflow, commit, git
---

## Không tự động tạo commit

### RULE-COMMIT-01: Không bao giờ tự tạo commit trừ khi được yêu cầu

KHÔNG tự động tạo git commit sau khi hoàn thành công việc. Chỉ tạo commit khi user yêu cầu rõ ràng (ví dụ: "tạo commit", "commit lại", "commit cho tôi").

**Sai:**
- Tự động chạy `git add` và `git commit` sau khi code xong
- Tạo commit ngay sau khi tạo file mới
- Commit mà không đợi user review

**Đúng:**
- Hoàn thành công việc và đợi user review
- Chỉ tạo commit khi user yêu cầu
