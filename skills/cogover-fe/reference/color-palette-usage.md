---
title: Luôn Sử dụng Configured Color Classes
impact: CRITICAL
impactDescription: Raw hex/rgb values trong class names bypass design token system, phá vỡ dark mode, và khiến việc rebrand color yêu cầu tìm và thay thế toàn bộ
tags: colors, tailwind, design-tokens, theme, dark-mode
---

## Luôn Sử dụng Configured Color Classes

Tất cả các giá trị color phải đến từ các classes được định nghĩa trong `tailwind.config.js`. Không bao giờ sử dụng raw hex, rgb, hoặc default color palette của Tailwind (ví dụ: `blue-500`) nếu project có custom color system.

**Trước khi sử dụng bất kỳ color class nào, kiểm tra `tailwind.config.js`** để xem tokens nào có sẵn.

### Sai:

```typescript
// ❌ Raw hex trong bracket notation
<div className="bg-[#1A73E8] text-[#202124]">

// ❌ Default palette của Tailwind bypass project tokens
<div className="bg-blue-500 text-gray-900">

// ❌ Inline style với color
<div style={{ color: '#5F6368' }}>
```

### Đúng:

```typescript
// ✅ Configured color tokens từ tailwind.config.js
<div className={cx("bg-primary-main text-text-primary")}>

// ✅ Semantic color names cho states
<p className={cx("text-danger-main")}>Error message</p>
<span className={cx("bg-success-subtle text-success-main")}>Active</span>
<div className={cx("border border-border-default")}>Card</div>
```

### Mẫu đặt tên color mong đợi trong config:

| Category | Example classes |
|----------|----------------|
| Primary | `text-primary-main`, `bg-primary-subtle`, `border-primary-main` |
| Text | `text-text-primary`, `text-text-secondary`, `text-text-muted`, `text-text-disabled` |
| Background | `bg-surface-default`, `bg-surface-raised`, `bg-surface-overlay` |
| Border | `border-border-default`, `border-border-strong` |
| Status | `text-danger-main`, `bg-warning-subtle`, `text-success-main`, `bg-info-subtle` |

Luôn sử dụng tên token semantic mô tả **mục đích**, không phải visual color. Điều này đảm bảo dark mode và theme overrides hoạt động tự động.
