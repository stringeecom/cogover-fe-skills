## Link & useNavigate (`uikit-link-navigate`)

Rule về import và sử dụng `Link` component và hook `useNavigate` để điều hướng trong app.

**Phạm vi áp dụng**: tất cả dự án frontend Cogover **trừ** dự án `id` và dự án `account` — 2 dự án này không có logic `appSlug` nên import/điều hướng theo chuẩn `react-router-dom` bình thường.

---

### RULE-LINK-NAV-01: Luôn import `Link` và `useNavigate` từ `@stringeecom/ui-kit`

Các dự án dùng chung ui-kit đã wrap lại `Link` và `useNavigate` để tự động prepend `appSlug` vào pathname. KHÔNG được import trực tiếp từ `react-router-dom` / `react-router` ở code ứng dụng — sẽ mất `appSlug`, dẫn tới điều hướng sai route.

```tsx
// ❌ SAI — mất appSlug
import { Link, useNavigate } from "react-router-dom";

// ✅ ĐÚNG
import { Link, useNavigate } from "@stringeecom/ui-kit";
```

Áp dụng cho cả:
- JSX: `<Link to="/records/123">...</Link>`
- Hook: `const navigate = useNavigate(); navigate("/records/123");`
- Truyền vào component khác: `<TextLink component={Link} to="/settings">Cài đặt</TextLink>`
- Truyền vào hook wrapper hoặc prop router integration

---

### RULE-LINK-NAV-02: Ngoại lệ duy nhất — khi truyền vào `StringeeUtilProvider`

`StringeeUtilProvider` là nơi wrapper `Link`/`useNavigate` của ui-kit được **khởi tạo** với `appSlug`. Tại file cấu hình provider này (và CHỈ tại file này) được phép import gốc từ `react-router-dom` để truyền vào.

```tsx
// ✅ Chỉ file setup provider được phép
import { Link, useNavigate } from "react-router-dom";
import { StringeeUtilProvider } from "@stringeecom/ui-kit";

<StringeeUtilProvider
    appSlug="crm"
    Link={Link}
    useNavigate={useNavigate}
    // ...other props
>
    {children}
</StringeeUtilProvider>
```

Mọi file khác vẫn phải import `Link`/`useNavigate` từ `@stringeecom/ui-kit`.

---

### RULE-LINK-NAV-03: Không tự prepend `appSlug` — wrapper đã làm

Khi đã import từ `@stringeecom/ui-kit`, wrapper tự gắn `appSlug` vào trước pathname. KHÔNG được tự thêm appSlug vào `to`/`pathname` → sẽ bị double prefix (`/crm/crm/records/123`).

```tsx
import { Link, useNavigate } from "@stringeecom/ui-kit";

// ❌ SAI — double prefix
<Link to={`/${appSlug}/records/123`}>Chi tiết</Link>
navigate(`/${appSlug}/records/123`);

// ✅ ĐÚNG — chỉ truyền pathname thuần
<Link to="/records/123">Chi tiết</Link>
navigate("/records/123");
```

---

### RULE-LINK-NAV-04: Ngoại lệ project — `id` và `account`

Dự án `id` và `account` KHÔNG có khái niệm `appSlug`, cũng không wrap `Link`/`useNavigate` qua ui-kit. Tại 2 dự án này:

- Import `Link`, `useNavigate` trực tiếp từ `react-router-dom`.
- Không áp dụng RULE-LINK-NAV-01, 02, 03.

```tsx
// ✅ Dự án id / account
import { Link, useNavigate } from "react-router-dom";
```

Các dự án khác (CRM, Omni, Voice, v.v.) vẫn tuân thủ RULE-LINK-NAV-01 → 03.
