---
title: Import nội bộ dùng path alias thay vì deep relative
impact: MEDIUM
impactDescription: Relative path sâu (nhiều hơn 2 cấp `..`) gây khó đọc, dễ lỗi khi refactor/di chuyển file, và làm rối review code
tags: import, path, alias, relative, src
---

## Import nội bộ dùng path alias thay vì deep relative

Khi import module/file nội bộ trong dự án, LUÔN dùng path alias dạng `src/foo/bar` thay cho relative path sâu dạng `../../../../foo/bar`. Cho phép relative path khi chỉ có tối đa 2 cấp `..`.

### RULE-IMPORT-PATH-01: Dùng path alias `src/...` thay cho deep relative

Nếu cần đi ngược lên **từ 3 cấp `..` trở lên**, PHẢI dùng path alias bắt đầu từ `src/...`. KHÔNG được dùng chuỗi `../../../...` dài.

**Sai:**
```tsx
// ❌ Relative path quá sâu (3 cấp `..` trở lên)
import { formatDate } from '../../../utils/formatDate';
import UserService from '../../../../services/UserService';
import { COLORS } from '../../../../../constants/colors';
```

**Đúng:**
```tsx
// ✅ Dùng path alias bắt đầu từ `src/...`
import { formatDate } from 'src/utils/formatDate';
import UserService from 'src/services/UserService';
import { COLORS } from 'src/constants/colors';
```

### RULE-IMPORT-PATH-02: Cho phép relative tối đa 2 cấp `..`

Relative path được phép khi độ sâu **không vượt quá 2 cấp `..`** (tức là `./...`, `../...`, `../../...`). Trong các trường hợp này, ưu tiên giữ relative để giữ tính gần gũi giữa các file liên quan trong cùng feature/module.

**Đúng:**
```tsx
// ✅ Relative 0–2 cấp `..` đều OK
import { helper } from './helper';
import { types } from '../types';
import { Child } from '../components/Child';
import { parentUtil } from '../../utils/parentUtil';
```

**Sai:**
```tsx
// ❌ Ngay khi vượt quá 2 cấp `..` → phải chuyển sang path alias
import { config } from '../../../config';           // → 'src/config'
import { api } from '../../../../services/api';     // → 'src/services/api'
```

### RULE-IMPORT-PATH-03: Không trộn lẫn style trong cùng một file

Trong cùng một file, giữ nhất quán: nếu đã dùng `src/...` cho một module ở cấp sâu, không quay lại dùng relative sâu cho module khác cùng cấp đó.

**Sai:**
```tsx
// ❌ Không nhất quán
import { formatDate } from 'src/utils/formatDate';
import { parseDate } from '../../../utils/parseDate';
```

**Đúng:**
```tsx
// ✅ Cùng style cho các module cùng khu vực
import { formatDate, parseDate } from 'src/utils';
// hoặc
import { formatDate } from 'src/utils/formatDate';
import { parseDate } from 'src/utils/parseDate';
```

### Tóm tắt quyết định nhanh

| Độ sâu relative | Cách dùng |
|---|---|
| `./...` | ✅ Relative |
| `../...` (1 cấp `..`) | ✅ Relative |
| `../../...` (2 cấp `..`) | ✅ Relative |
| `../../../...` (≥ 3 cấp `..`) | ❌ Dùng `src/...` |

> Lưu ý: Rule này chỉ áp dụng cho import nội bộ trong dự án. Import từ package (ví dụ `@stringeecom/ui-kit`) tuân theo `uikit-import-source`.
