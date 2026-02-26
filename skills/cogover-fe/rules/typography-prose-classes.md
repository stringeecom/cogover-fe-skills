---
title: Sử dụng prose-<type> Classes cho Typography
impact: CRITICAL
impactDescription: Việc set font-size, font-weight, và line-height với arbitrary Tailwind values gây ra visual inconsistency và khiến việc thay đổi typography trên toàn design-system trở nên không thể
tags: typography, prose, font-size, font-weight, line-height, tailwind
---

## Sử dụng prose-\<type\> Classes cho Typography

Tất cả styling về font-size, font-weight, và line-height phải sử dụng utility classes `prose-<type>`. Các classes này được định nghĩa trong `theme.extend.typography` trong `tailwind.config.js`. Không bao giờ sử dụng `text-[...]`, `font-[...]`, hoặc `leading-[...]` arbitrary values cho typography.

**Trước khi viết bất kỳ prose class nào, đọc `tailwind.config.js`** để xem các types nào có sẵn. Không tự sáng tạo tên type mới.

### Quy ước đặt tên type

Các types có numeric suffix (`body1`/`body2`, `label1`/`label2`, v.v.) chia sẻ cùng `font-size` và `line-height`. Chúng chỉ khác nhau ở `font-weight`:
- **Suffix `2`** → normal weight (default body text weight)
- **Suffix `1`** → heavier weight (bold/semi-bold emphasis)

**Default body text phải luôn sử dụng `prose-body2`.**

### Sai:

```typescript
// ❌ Arbitrary font-size, raw font-weight, raw line-height
<p className="text-[14px] font-normal leading-[1.5]">Description</p>
<h2 className="text-[24px] font-semibold leading-[1.2]">Title</h2>
<span className="text-sm font-medium">Label</span>
```

### Đúng:

```typescript
// ✅ prose-<type> classes — đọc available types từ tailwind.config.js
<p className={cx("prose-body2 text-text-secondary")}>Description</p>
<h2 className={cx("prose-h2 text-text-primary")}>Title</h2>
<span className={cx("prose-label1 text-text-muted")}>Label</span>

// ✅ body1 vs body2 — cùng size/line-height, khác weight
<p className={cx("prose-body2")}>Normal weight paragraph</p>
<p className={cx("prose-body1")}>Bold-emphasis paragraph</p>
```

### Không thêm types mới:

```typescript
// ❌ Không tự sáng tạo prose types không có trong tailwind.config.js
<p className="prose-display3">Hero text</p>  // nếu display3 không được định nghĩa
```

Nếu thực sự cần một typography style mới, hãy thêm nó vào `tailwind.config.js` dưới `theme.extend.typography` — không sử dụng arbitrary values như một workaround.
