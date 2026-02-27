---
title: Import tất cả component từ @stringeecom/ui-kit
impact: HIGH
impactDescription: Import sai nguồn dẫn đến bundle size lớn hơn, không đồng nhất codebase, và có thể gây lỗi khi build
tags: import, ui-kit, module, package
---

## Import tất cả component từ @stringeecom/ui-kit

Tất cả các component, hook, utility trong ui-kit phải được import từ package `@stringeecom/ui-kit`.

### RULE-IMPORT-01: Import component từ `@stringeecom/ui-kit`

Luôn import component từ `@stringeecom/ui-kit`. KHÔNG import trực tiếp từ đường dẫn nội bộ hoặc từ nguồn khác.

**Sai:**
```tsx
// ❌ Import từ đường dẫn nội bộ
import { Button } from 'ui-kit/src/components/Button';
import { Select } from '../../ui-kit/src/components/Select';
import { Modal } from '../../../packages/ui-kit/components/Modal';
```

**Đúng:**
```tsx
// ✅ Import từ @stringeecom/ui-kit
import { Button, Modal, Select } from '@stringeecom/ui-kit';
```

### RULE-IMPORT-02: Gộp import từ cùng package

Khi sử dụng nhiều component từ ui-kit, gộp tất cả vào một câu lệnh import duy nhất.

**Sai:**
```tsx
// ❌ Nhiều câu lệnh import riêng lẻ
import { Button } from '@stringeecom/ui-kit';
import { Modal } from '@stringeecom/ui-kit';
import { Select } from '@stringeecom/ui-kit';
```

**Đúng:**
```tsx
// ✅ Gộp thành một câu lệnh import
import { Button, Modal, Select } from '@stringeecom/ui-kit';
```

### RULE-IMPORT-03: Import hook và utility từ `@stringeecom/ui-kit`

Các hook và utility cũng phải được import từ `@stringeecom/ui-kit`.

**Sai:**
```tsx
// ❌ Import hook từ đường dẫn nội bộ
import { useResponsive } from 'ui-kit/src/hooks/useResponsive';
import { cx } from 'ui-kit/src/utils/cx';
```

**Đúng:**
```tsx
// ✅ Import hook và utility từ @stringeecom/ui-kit
import { useResponsive, cx } from '@stringeecom/ui-kit';
```
