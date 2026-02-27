---
title: Text mới phải viết bằng tiếng Việt
impact: HIGH
impactDescription: Viết text bằng tiếng Anh hoặc dùng i18n key ngay từ đầu gây khó review và không đúng quy trình làm việc
tags: text, vietnamese, i18n, workflow
---

## Text mới phải viết bằng tiếng Việt

### RULE-TEXT-01: Tất cả text mới tạo ra phải là tiếng Việt

Khi tạo mới hoặc chỉnh sửa UI, tất cả các text hiển thị (label, placeholder, message, title, description...) phải được viết bằng tiếng Việt trực tiếp. KHÔNG tự động chuyển sang i18n key.

**Sai:**
```tsx
// ❌ Tự động dùng i18n key khi chưa được yêu cầu
<Button>{t('common.save')}</Button>
<TextField placeholder={t('user.enterName')} />
<span>{t('order.totalAmount')}</span>
```

**Đúng:**
```tsx
// ✅ Viết text tiếng Việt trực tiếp
<Button>Lưu</Button>
<TextField placeholder="Nhập tên" />
<span>Tổng tiền</span>
```

### RULE-TEXT-02: Chỉ dịch sang i18n khi được yêu cầu sử dụng skill ui-i18n

Việc chuyển đổi text tiếng Việt sang i18n key chỉ được thực hiện khi có yêu cầu rõ ràng sử dụng skill `ui-i18n`. Không tự ý dịch hoặc thay thế text tiếng Việt bằng i18n key.
