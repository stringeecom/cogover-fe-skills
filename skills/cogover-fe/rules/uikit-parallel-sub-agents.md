---
title: Phân tích và tách công việc thành sub agent song song
impact: MEDIUM
impactDescription: Không tách công việc dẫn đến thời gian thực hiện lâu hơn cần thiết
tags: workflow, sub-agent, parallel, performance
---

## Phân tích và tách công việc thành sub agent song song

### RULE-AGENT-01: Phân tích yêu cầu trước khi thực hiện

Trước khi bắt đầu làm bất kỳ yêu cầu nào, phải phân tích xem có thể tách thành nhiều công việc nhỏ không phụ thuộc vào nhau hay không.

**Quy trình:**
1. Đọc và hiểu toàn bộ yêu cầu
2. Xác định các công việc con
3. Phân tích sự phụ thuộc giữa các công việc con
4. Nếu tách được từ **2 công việc nhỏ trở lên** mà không phụ thuộc nhau thì tự động sử dụng sub agent để thực hiện song song

### RULE-AGENT-02: Tự động dùng sub agent khi có thể

Khi xác định được các công việc độc lập, tự động sử dụng sub agent (Task tool) để thực hiện song song mà không cần hỏi lại.

**Ví dụ tách được:**
- Tạo 3 component mới không liên quan nhau -> 3 sub agent song song
- Sửa bug ở 2 file khác nhau, không phụ thuộc -> 2 sub agent song song
- Thêm rule cho 5 component khác nhau -> 5 sub agent song song

**Ví dụ KHÔNG tách được:**
- Component B import từ component A -> phải làm tuần tự
- Sửa type definition rồi mới sửa component dùng type đó -> phải làm tuần tự
