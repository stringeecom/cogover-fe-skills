# Unit Test - @stringeecom/ui-kit

## Nguyên tắc tuyệt đối (đọc trước tiên)

Các rule này KHÔNG có ngoại lệ. Vi phạm = sửa lại, không giải thích:

1. **KHÔNG cast type trong mock** — mọi mock data / mock props / mock response / mock handler / mock context phải khai báo đầy đủ theo đúng type. Cấm `as unknown as T`, `as any`, `as T`, `{} as T`, `<T>{}`. Type lớn → dùng **factory function** có default đủ type + `Partial<T>` override. Xem section "Mock Data" bên dưới.
2. **KHÔNG mock component con, hook nội bộ, util nội bộ, `@stringeecom/ui-kit`, `@fortawesome/*`** — render thật toàn bộ tree.
3. **KHÔNG `vi.mock('src/apis/...')`** — mock API qua MSW handler mặc định trong `src/__mocks__/handlers/`.
4. **KHÔNG `fireEvent`** — luôn dùng `userEvent`.
5. **KHÔNG `npx vitest`** — luôn qua `npm run test` / `npm run coverage`.
6. **Coverage (TUYỆT ĐỐI, KHÔNG NEGOTIATE)**: **lines = 100%** VÀ **branches = 100%** cho mỗi file source. KHÔNG chấp nhận bất kỳ tỷ lệ nào thấp hơn — thiếu 1 line hoặc 1 branch = chưa xong. Chỉ được phép < 100% khi dùng `/* v8 ignore */` kèm lý do defensive rõ ràng (xem section "Yêu cầu Coverage").
7. **BẮT BUỘC FEEDBACK khi phát hiện logic bất hợp lý** — trong quá trình viết test, nếu thấy bất kỳ logic nào trong code nguồn **có vẻ sai, mâu thuẫn, hoặc không hợp lý** (điều kiện ngược, dependency thiếu/thừa, edge case không handle, return value lạ, side-effect không rõ ràng, naming không khớp behavior...) → **DỪNG NGAY, KHÔNG viết test theo logic đó**, báo lại cho user kiểm tra xem code có bị bug không. Tuyệt đối tránh việc "viết test pass theo logic sai" — điều đó sẽ lock bug vào codebase và khiến test mất ý nghĩa. Xem section "Feedback khi phát hiện logic bất hợp lý" bên dưới.

## Tech Stack

- **Test framework**: Vitest (global APIs: `describe`, `it`, `expect`, `vi`)
- **Component testing**: `@testing-library/react` (`render`, `screen`, `waitFor`)
- **User interaction**: `@testing-library/user-event`
- **Hook testing**: `renderHook` from `@testing-library/react`
- **API mocking**: MSW v2 (Mock Service Worker) via `msw/node`
- **Environment**: jsdom, setup file tại `src/__test__/setup.ts`

## Feedback khi phát hiện logic bất hợp lý (CRITICAL)

**Nguyên tắc:** Unit test là để **verify behavior đúng**, KHÔNG phải để "ép test pass với code đang có". Nếu code nguồn có logic sai, việc viết test pass theo logic sai = khóa bug vào codebase + làm test trở thành regression guard cho bug.

### Các dấu hiệu cần feedback ngay

Trong quá trình đọc code để viết test, nếu gặp bất kỳ dấu hiệu nào dưới đây → **DỪNG viết test, báo ngay cho user**:

1. **Điều kiện ngược / sai dấu**: `if (!isValid)` nhưng comment/naming gợi ý logic ngược lại; `>` thay vì `>=` ở boundary; early return nhầm branch.
2. **Dependency của hook không đúng**: `useEffect`/`useMemo`/`useCallback` thiếu dependency (sẽ stale) hoặc thừa dependency (chạy lại không cần thiết) — xem thêm RULE-TQ-02 về `select` trong TanStack Query.
3. **Edge case không handle**: không check `null`/`undefined`/empty array/array rỗng trước khi truy cập, không handle lỗi API trả về, divide by zero...
4. **Return value lạ / không nhất quán**: hàm khi thì return `null`, khi thì return `undefined`, khi thì throw — không theo contract rõ ràng.
5. **Side-effect không rõ ràng**: function pure lẽ ra không mutate nhưng lại mutate argument; hook gọi setState trong render.
6. **Naming không khớp behavior**: tên hàm `isActive` nhưng return ngược lại; biến `userId` nhưng chứa `userName`.
7. **Race condition tiềm ẩn**: không cleanup subscription/timer trong `useEffect`, không cancel request cũ khi input thay đổi.
8. **Logic duplicate mâu thuẫn**: 2 nơi cùng compute 1 giá trị nhưng công thức khác nhau.
9. **Type assertion che bug**: `as Type`, `as any`, `!` non-null assertion ở chỗ runtime có thể thực sự là null.
10. **Comment/JSDoc không khớp implementation**: docstring nói "returns true if X" nhưng code return false trong case X.

### Quy trình khi phát hiện

1. **DỪNG** viết test cho file/function đó ngay.
2. **Ghi lại** chính xác: file path, line number, đoạn code nghi ngờ, lý do nghi ngờ, behavior kỳ vọng vs behavior hiện tại.
3. **Báo cho user** theo format:

   ```
   ⚠️ Nghi ngờ logic bất hợp lý — cần kiểm tra trước khi viết test

   File: src/xxx/yyy.ts:42
   Code:
     if (user.age > 18) { return 'adult' }

   Nghi ngờ: Boundary có vẻ sai — theo convention "adult từ 18 tuổi"
   thì phải là `>= 18`. Hiện tại user đúng 18 tuổi sẽ không được tính là adult.

   → Đây là bug hay intended? Nếu bug, tôi fix code trước rồi mới viết test.
   ```

4. **CHỜ** user confirm:
   - User bảo "đó là bug, fix đi" → fix code trước, rồi mới viết test theo behavior đúng.
   - User bảo "đó là intended" → viết test theo logic hiện tại, kèm comment giải thích tại sao boundary đó là intended (để tránh người sau nghi ngờ lại).
   - User bảo "để sau, cứ skip" → thêm `it.skip` kèm comment `// TODO: confirm behavior` và chuyển sang file khác.

### Sai — tuyệt đối không làm

```typescript
// ❌ Thấy boundary nghi vấn nhưng tự viết test theo logic hiện tại
// mà không hỏi → lock bug vào test
it("should return 'adult' when age > 18", () => {
  expect(getAgeGroup({ age: 19 })).toBe("adult");
  expect(getAgeGroup({ age: 18 })).toBe("minor"); // ← test này khoá bug!
});
```

### Đúng

```typescript
// ✅ Feedback cho user trước, confirm xong mới viết test
// (Sau khi user confirm "đó là bug, cần fix"):
// → Fix code: if (user.age >= 18)
// → Viết test theo behavior đúng:
it("should return 'adult' when age is 18 or above", () => {
  expect(getAgeGroup({ age: 18 })).toBe("adult");
  expect(getAgeGroup({ age: 19 })).toBe("adult");
});
it("should return 'minor' when age is below 18", () => {
  expect(getAgeGroup({ age: 17 })).toBe("minor");
});
```

## Quy trình viết test

### Bước 0: Khảo sát cấu trúc thư mục (BẮT BUỘC)

**Quy tắc cốt lõi: Khi được yêu cầu viết unit test cho 1 folder/module/feature, KHÔNG được viết ngay vào file user chỉ định. PHẢI khảo sát toàn bộ cấu trúc thư mục trước, rồi viết test theo thứ tự leaf-first (bottom-up).**

#### Vì sao phải làm vậy?

- Component cha render thật component con (theo rule "KHÔNG mock component"). Nếu component con chưa có test và còn bug → test cha fail vì lý do không liên quan tới cha → khó debug.
- Utils/hooks được nhiều component dùng chung → test utils trước sẽ phát hiện bug sớm, tránh viết lại nhiều lần khi fix.
- Leaf file (utils, helpers, pure functions) coverage dễ đạt 100% nhất → làm trước để build momentum.

#### Các bước khảo sát

1. **Map toàn bộ cây thư mục** của phần cần test bằng `Glob` hoặc `list_dir`:
   ```
   Glob: src/components/Foo/**/*.{ts,tsx}
   ```
   Liệt kê mọi file source (loại trừ file test, `index.ts` re-export thuần).

2. **Phân loại từng file** theo độ sâu dependency:
   - **Tier 0 — Leaf (không dependency nội bộ)**: `utils/*.ts`, `constants.ts`, `types.ts`, pure helpers. Không import từ file khác trong cùng folder.
   - **Tier 1 — Hooks**: `hooks/*.ts`. Có thể import từ Tier 0, mock API nếu cần.
   - **Tier 2 — Leaf components**: component nhỏ không chứa component con nội bộ (chỉ dùng `@stringeecom/ui-kit`). Có thể import Tier 0 + Tier 1.
   - **Tier 3 — Container components**: component chứa Tier 2 bên trong.
   - **Tier 4 — Page/Root**: file top-level ráp toàn bộ.

3. **Kiểm tra test đã tồn tại** bằng `Glob` trong `src/__test__/` mirror path → bỏ qua file đã có test, tránh duplicate.

4. **Lập danh sách thứ tự viết test** (bằng `TaskCreate` nếu >= 3 file): Tier 0 → Tier 1 → Tier 2 → Tier 3 → Tier 4. Trong cùng tier, viết file nào cũng được.

5. **Viết test tuần tự theo thứ tự trên**, mỗi file:
   - Viết test → chạy test (Bước 1 "Workflow tiết kiệm token") → verify pass
   - Check coverage (Bước 2) → bổ sung case tới khi **lines = 100% VÀ branches = 100%**
   - Mới chuyển sang file tier tiếp theo

#### Ví dụ

User yêu cầu: *"Viết unit test cho `src/components/Chat/ChatBox/`"*

Cây thư mục:
```
src/components/Chat/ChatBox/
├── index.tsx                       ← Tier 3 (import ChatInput, MessageList, useChatSocket)
├── ChatInput.tsx                   ← Tier 2 (leaf component)
├── MessageList.tsx                 ← Tier 2 (leaf component)
├── hooks/
│   ├── useChatSocket.ts            ← Tier 1 (hook, gọi API)
│   └── useMessageScroll.ts         ← Tier 1 (hook thuần)
└── utils/
    └── messageFormatter.ts         ← Tier 0 (pure function)
```

Thứ tự viết test ĐÚNG:
1. `utils/messageFormatter.ts` (Tier 0)
2. `hooks/useMessageScroll.ts`, `hooks/useChatSocket.ts` (Tier 1)
3. `ChatInput.tsx`, `MessageList.tsx` (Tier 2)
4. `index.tsx` (Tier 3)

❌ SAI: viết ngay test cho `index.tsx` khi các file con chưa có test.

### Bước 1: Đọc code, tìm hiểu nghiệp vụ

Đọc toàn bộ file source để hiểu:
- Nghiệp vụ của component/hook/hàm đang làm gì — input, output, side effects
- Các functions/hooks/components được export
- Dependencies và imports (đặc biệt là các API call)
- Logic branching, edge cases
- Types và interfaces

Sau khi hiểu code, quyết định flow tiếp theo:

- **Không cần mock API** (pure function, hook thuần, component không gọi API) → nhảy thẳng sang **Bước 2** viết test luôn.
- **Cần mock API** → sang **Bước 3** phân tích các API cần mock, viết/mở rộng handler trong `src/__mocks__/handlers/` trước, **rồi mới** quay lại **Bước 2** viết test.

### Bước 2: Xác định loại test

#### A. Pure Function Test (`.test.ts`)

Dành cho: `src/utils/*.ts`, helper functions, pure logic

```typescript
import { describe, expect, it } from "vitest";
import { functionName } from "../../utils/fileName";

describe("functionName", () => {
    // Pattern 1: Input/Output array cho nhiều test cases
    const io: { input: InputType; output: OutputType }[] = [
        { input: ..., output: ... },
        { input: ..., output: ... },
    ];

    io.forEach(({ input, output }, index) => {
        it(`Case ${index + 1}: mô tả ngắn`, () => {
            const result = functionName(input);
            expect(result).toEqual(output);
        });
    });

    // Pattern 2: Individual test cases
    it("should handle edge case", () => {
        expect(functionName(null)).toBe(defaultValue);
    });
});
```

**Assertions phổ biến:**
- `expect(result).toBe(value)` - strict equality
- `expect(result).toEqual(value)` - deep equality cho objects/arrays
- `expect(result).toBeTruthy()` / `toBeFalsy()`
- `expect(result).toBeNull()` / `toBeUndefined()`
- `expect(result).toContain(item)` - array/string contains
- `expect(result).toHaveLength(n)`
- `expect(() => fn()).toThrow()`

#### B. Hook Test (`.test.tsx`)

Dành cho: `src/hooks/*.ts`, `src/components/**/hooks/*.ts`

```typescript
import { renderHook, waitFor } from "@testing-library/react";
import { describe, expect, it } from "vitest";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactNode } from "react";
import { MemoryRouter } from "react-router-dom";
import { renderWrapper } from "../../testUtils"; // adjust path

// Wrapper chuẩn cho hooks cần providers
const createWrapper = () => {
    const queryClient = new QueryClient({
        defaultOptions: { queries: { retry: false } },
    });

    return ({ children }: { children: ReactNode }) =>
        renderWrapper({
            children: (
                <QueryClientProvider client={queryClient}>
                    <MemoryRouter initialEntries={["/"]}>{children}</MemoryRouter>
                </QueryClientProvider>
            ),
        });
};

describe("useHookName", () => {
    it("should return expected value", () => {
        const { result } = renderHook(() => useHookName(params), {
            wrapper: createWrapper(),
        });

        expect(result.current.value).toBe(expected);
    });

    // Async hook (React Query, etc.)
    it("should fetch data", async () => {
        const { result } = renderHook(() => useHookName(), {
            wrapper: createWrapper(),
        });

        await waitFor(() => {
            expect(result.current.data).toBeDefined();
        });
    });
});
```

**Wrapper tối giản** (hook không cần router/providers):
```typescript
const { result } = renderHook(() => useHookName(params), {
    wrapper: ({ children }) => (
        <QueryClientProvider client={new QueryClient()}>
            {children}
        </QueryClientProvider>
    ),
});
```

#### C. Component Test (`.test.tsx`)

Dành cho: `src/components/**/*.tsx`

```typescript
import { render, screen, waitFor } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { describe, expect, it, vi } from "vitest";
import { renderComponent } from "../../testUtils"; // adjust path

describe("ComponentName", () => {
    // Component cần providers (hầu hết cases)
    it("should render correctly", () => {
        renderComponent(<ComponentName prop1="value" />);
        expect(screen.getByText("expected text")).toBeVisible();
    });

    // User interaction
    it("should handle click", async () => {
        const { user } = renderComponent(<ComponentName />);
        
        await user.click(screen.getByRole("button"));
        
        expect(screen.getByText("result")).toBeVisible();
    });

    // Component đơn giản không cần providers
    it("should render with props", () => {
        const onChange = vi.fn();
        render(<SimpleComponent value="test" onChange={onChange} />);
        
        expect(screen.getByDisplayValue("test")).toBeInTheDocument();
    });

    // Callback test
    it("should call onChange", async () => {
        const onChange = vi.fn();
        const { user } = renderComponent(
            <ComponentName onChange={onChange} />
        );
        
        await user.type(screen.getByRole("textbox"), "new value");
        
        await waitFor(() => {
            expect(onChange).toHaveBeenCalledWith("new value");
        });
    });
});
```

**Component queries ưu tiên (theo testing-library):**
1. `getByRole` - buttons, textboxes, headings
2. `getByText` - visible text content
3. `getByTestId` - data-testid attribute (fallback)
4. `getByDisplayValue` - input values
5. `container.querySelector` - CSS selectors (last resort)

**Thêm class / data-testid vào source code:**

Nếu cần thêm `class` hoặc `data-testid` vào source component để phục vụ viết test thì **được phép tự thêm**. Tuy nhiên **KHÔNG được tự ý xoá** `class` hoặc `data-testid` đã có sẵn mà không được user cho phép.

**User interaction (BẮT BUỘC dùng userEvent, KHÔNG dùng fireEvent):**

Luôn dùng `userEvent` từ `@testing-library/user-event` để giả lập hành vi người dùng. KHÔNG dùng `fireEvent` vì nó chỉ dispatch DOM event đơn lẻ, không mô phỏng đúng hành vi thật của browser (focus, pointer, keyboard sequence...).

```typescript
// ❌ SAI: fireEvent
import { fireEvent } from "@testing-library/react";
fireEvent.click(button);
fireEvent.change(input, { target: { value: "text" } });

// ✅ ĐÚNG: userEvent
import userEvent from "@testing-library/user-event";

// Cách 1: Lấy từ renderComponent (đã setup sẵn)
const { user } = renderComponent(<MyComponent />);
await user.click(button);
await user.type(input, "text");

// Cách 2: Setup thủ công (khi dùng render thay vì renderComponent)
const user = userEvent.setup();
render(<MyComponent />);
await user.click(button);
await user.type(input, "text");
```

### Bước 3: Mock API (nếu cần)

**QUAN TRỌNG: KHÔNG dùng `vi.mock()` để mock module API (`src/apis/*`). Dùng MSW intercept ở tầng network** để test chạy thật từ component → hook → API call → MSW trả response.

**Thứ tự làm việc bắt buộc khi có mock API**:
1. **Phân tích API cần mock** — list ra tất cả endpoint/service mà component/hook sẽ gọi, từng API cần những response nào (happy path, empty, error, filter theo param…).
2. **Viết hoặc mở rộng handler** trong `src/__mocks__/handlers/` cho đủ các response cần thiết (thêm nhánh if/else theo params nếu cần). Chưa viết test.
3. **Mới viết test** dùng handler vừa viết. Không vừa viết test vừa nghĩ mock.

**Nguyên tắc cốt lõi: Ưu tiên viết handler mặc định trong `src/__mocks__/handlers/`. Khi cần nhiều response khác nhau cho cùng 1 API, truyền params khác nhau từ test và xử lý if/else trong handler để trả về response tương ứng. HẠN CHẾ dùng `server.use()` kèm `{ once: true }` trong file test.**

Vì sao?
- Handler mặc định tập trung 1 chỗ → dễ maintain, dễ tái sử dụng giữa nhiều test files.
- Override trong test body làm test case rối, trùng lặp logic mock, khó reason về thứ tự handler.
- Logic if/else theo params trong handler mô phỏng đúng hành vi backend thật (cùng endpoint, response khác nhau theo input).

#### Kiến trúc API mock trong project

Project dùng **service-based routing** — tất cả requests gửi tới 1 endpoint duy nhất (ví dụ `multiChannelUri.index`), phân biệt service bằng header `X_SERVICE_KEY`.

```
src/__mocks__/
├── _mock.ts              # Helper: _mockApiSuccess, _mockApiError, _mockSuccessServiceApi
├── handlers/
│   └── omni.handler.ts   # MSW handlers cho omni-channel APIs
│   └── ...               # Các handler khác
├── multiChannel.ts       # Mock data cho MultiChannel
├── routing.ts            # Mock data cho Routing
└── sampleMessage.ts      # Mock data cho SampleMessage
```

#### Flow hoạt động

1. **MSW chặn HTTP request** tại endpoint (POST)
2. **Router function** đọc `X_SERVICE_KEY` từ header → xác định service number
3. **Lookup routes map** → gọi hàm handler tương ứng
4. **Handler trả về** `HttpResponse.json(_mockApiSuccess(data))`

#### Cách viết handler mới

Handler mặc định đặt trong `src/__mocks__/handlers/`. Khi test cần nhiều response khác nhau cho cùng 1 API → **không** override trong test body, mà **truyền params khác nhau từ test** (ví dụ `id`, `slug`, `keyword`, `status` trong `body`) rồi xử lý if/else ngay trong handler để map params → response.

```typescript
import { http, HttpHandler, HttpResponse } from "msw";
import { _mockApiSuccess, _mockApiError } from "../_mock";

// 1. Chuẩn bị các biến thể mock data
const MOCK_ITEMS = [
    { id: "1", name: "Item 1", status: "active" },
    { id: "2", name: "Item 2", status: "inactive" },
];

// 2. Handler branch theo params trong body
const listHandler = (body: { keyword?: string; status?: string }) => {
    // Case: search không ra kết quả
    if (body.keyword === "__empty__") {
        return HttpResponse.json(_mockApiSuccess({ total: 0, records: [] }));
    }
    // Case: filter theo status
    if (body.status) {
        const records = MOCK_ITEMS.filter((item) => item.status === body.status);
        return HttpResponse.json(_mockApiSuccess({ total: records.length, records }));
    }
    // Case mặc định: trả full list
    return HttpResponse.json(_mockApiSuccess({ total: MOCK_ITEMS.length, records: MOCK_ITEMS }));
};

const detailHandler = (body: { id: string }) => {
    // Case: id không tồn tại → error
    if (body.id === "__not_found__") {
        return HttpResponse.json(_mockApiError("Item not found", 1));
    }
    const item = MOCK_ITEMS.find((i) => i.id === body.id) ?? MOCK_ITEMS[0];
    return HttpResponse.json(_mockApiSuccess(item));
};

// 3. Routes map: service number → handler function
const routes = {
    [myServices.list]: (body) => listHandler(body as never),
    [myServices.detail]: (body) => detailHandler(body as never),
    [myServices.create]: () => HttpResponse.json(_mockApiSuccess(null)),
} as Record<number, (body: unknown, service: number) => StrictResponse<JsonBodyType>>;

// 4. Router function: đọc header → dispatch đến handler
async function serviceRouter({ request }: ResolverParams) {
    const payload = (await request.json()) as { body: unknown };
    const service = Number(request.headers.get(X_SERVICE_KEY));
    if (!service) return HttpResponse.json({ message: "service not found" }, { status: 404 });
    const fn = routes[service];
    if (!fn) return HttpResponse.json({ message: "route not found" }, { status: 404 });
    return fn(payload.body, service);
}

// 5. Export MSW handler
export const myHandler: HttpHandler[] = [http.post(myUri.index, serviceRouter)];
```

Test sử dụng handler này bằng cách truyền params khác nhau vào component/hook:

```typescript
// Case mặc định — không cần override
it("should render full list", async () => {
    renderComponent(<ItemList />);
    await waitFor(() => expect(screen.getByText("Item 1")).toBeVisible());
});

// Case empty — truyền keyword sentinel đã được handler handle
it("should show empty state when search returns no result", async () => {
    const { user } = renderComponent(<ItemList />);
    await user.type(screen.getByRole("textbox"), "__empty__");
    await waitFor(() => expect(screen.getByText("No data")).toBeVisible());
});

// Case error — truyền id sentinel mà handler map sang error response
it("should show error toast when item not found", async () => {
    renderComponent(<ItemDetail id="__not_found__" />);
    await waitFor(() => expect(screen.getByText("Item not found")).toBeVisible());
});
```

#### Response helpers có sẵn (`src/__mocks__/_mock.ts`)

```typescript
// Success: { r: 0, msg: "success", data: data, meta: meta }
_mockApiSuccess(data, meta?)

// Error: { r: number, msg: "error message" }
_mockApiError(msg, r?)

// Service API success (qua recordUri.index):
// { serviceVersion: 1, service, type, body: { r: 0, msg: "success", data } }
_mockSuccessServiceApi({ service, type, data })
```

#### Override handler trong test body (chỉ khi thật sự cần)

**Hạn chế tối đa. Ưu tiên mở rộng handler mặc định theo params như trên.** Chỉ override trong test body khi:

- Case rất cá biệt chỉ xuất hiện trong 1 test, không muốn làm bẩn handler mặc định.
- Cần mô phỏng lỗi network / timeout / status code đặc biệt mà handler mặc định không model.
- Không có cách nào truyền params từ component để handler mặc định phân biệt case.

Nếu rơi vào 1 trong các trường hợp trên, dùng `server.use()` kèm `{ once: true }`:

```typescript
import { http, HttpResponse } from "msw";
import { server } from "../../setup"; // adjust path

it("should handle 500 network error", async () => {
    server.use(
        http.post("/api/endpoint", () => {
            return new HttpResponse(null, { status: 500 });
        }, { once: true })  // chỉ chạy 1 lần, sau đó revert về handler mặc định
    );

    renderComponent(<MyComponent />);
    // ... assertions
});
```

Trước khi override, luôn cân nhắc: *có thể thêm 1 nhánh if/else trong handler mặc định và truyền sentinel params từ component không?* Nếu có → làm cách đó.

#### Sử dụng mock utilities có sẵn (`src/__mocks__/utils.ts`)

```typescript
import {
    mockGetObjectDetailBySlugApi,
    mockGetRecordListApi,
    mockGetLayout,
    mockGetPersonnelListApi,
    mockFilterList,
} from "../../__mock__/utils"; // adjust path

it("should load records", async () => {
    mockGetRecordListApi([{ id: "1", name: "Record 1" }]);

    renderComponent(<RecordList objectSlug="contact" />);

    await waitFor(() => {
        expect(screen.getByText("Record 1")).toBeVisible();
    });
});
```

#### Quy tắc mock API

- **KHÔNG `vi.mock('src/apis/...')`** — dùng MSW handler thay thế
- **Ưu tiên handler mặc định** trong `src/__mocks__/handlers/` — tự động load qua `setup.ts`
- **Khi cần nhiều response khác nhau cho cùng 1 API → truyền params khác nhau từ test và viết if/else trong handler**, KHÔNG override trong test body
- **HẠN CHẾ `server.use()` + `{ once: true }`** trong file test — chỉ dùng cho case cá biệt (network error, status code lạ) không thể model bằng params
- **`server.use()`** chỉ dùng trong test body, KHÔNG dùng trong `beforeAll`

## File Structure Convention

**Quy tắc cốt lõi: Cấu trúc folder trong `src/__test__/` phải mirror 1:1 với cấu trúc folder source, chỉ thay extension `.tsx` → `.test.tsx`, `.ts` → `.test.ts`.**

Ví dụ source:
```
src/components/Chat/
├── ChatBox/
│   ├── index.tsx
│   ├── ChatInput.tsx
│   ├── MessageList.tsx
│   └── hooks/
│       ├── useChatSocket.ts
│       └── useMessageScroll.ts
├── ChatSidebar/
│   ├── index.tsx
│   └── ConversationItem.tsx
└── utils/
    └── messageFormatter.ts
```

Test tương ứng:
```
src/__test__/components/Chat/
├── ChatBox/
│   ├── index.test.tsx
│   ├── ChatInput.test.tsx
│   ├── MessageList.test.tsx
│   └── hooks/
│       ├── useChatSocket.test.ts
│       └── useMessageScroll.test.ts
├── ChatSidebar/
│   ├── index.test.tsx
│   └── ConversationItem.test.tsx
└── utils/
    └── messageFormatter.test.ts
```

Các file hỗ trợ chung:
```
src/__test__/
├── setup.ts                          # MSW server + global setup
├── testUtils.tsx                     # renderComponent, renderWrapper, helpers
```

**Quy tắc đặt file:**
- **Mirror 1:1**: Mỗi file source có đúng 1 file test tương ứng, cùng vị trí trong cây thư mục, chỉ thêm `.test` trước extension
- `src/components/Foo/Bar.tsx` → `src/__test__/components/Foo/Bar.test.tsx`
- `src/components/Foo/hooks/useBar.ts` → `src/__test__/components/Foo/hooks/useBar.test.ts`
- `src/components/Foo/utils/helper.ts` → `src/__test__/components/Foo/utils/helper.test.ts`
- `src/utils/[fileName].ts` → `src/__test__/utils/[fileName].test.ts`
- **KHÔNG** gom nhiều file source vào 1 file test chung (vd: KHÔNG tạo `index.test.tsx` rồi test cả `ChatInput` + `MessageList` trong đó)
- **KHÔNG** tạo thêm folder trung gian không có trong source (vd: KHÔNG tạo `__test__/components/Chat/ChatBox/tests/`)

## MSW Setup (đã cấu hình sẵn)

**`src/__test__/setup.ts`** - Auto-run trước mỗi test:
- `beforeAll`: `server.listen()` + mock browser APIs (scrollIntoView, scrollTo, requestIdleCallback)
- `afterEach`: `server.resetHandlers()` + `cleanup()`
- `afterAll`: `server.close()`

**`src/__mock__/index.ts`** - Handler registry:
- config, department, personnel, object, record, notification, activity, layout

**`src/__mock__/utils.ts`** - Dynamic mock helpers:
- `mockGetObjectDetailBySlugApi(object)` - Override object fetch
- `mockGetObjectDetailApi(object)` - Override object list
- `mockGetRecordListApi(records, meta?)` - Override record list
- `mockGetPersonnelListApi(records)` - Override personnel
- `mockGetPositionListApi(positions)` - Override positions
- `mockGetLayout(objectSlug, functionName, response)` - Override layout
- `mockGetLayoutById(objectSlug, layoutId, response)` - Override layout by ID
- `mockFilterList(filters)` - Override filter list

All dùng `{ once: true }` → chỉ run 1 lần rồi revert.

## Test Utilities (đã có sẵn)

**`src/__test__/testUtils.tsx`**:

| Function | Mục đích |
|----------|----------|
| `renderWrapper({ children, ...props })` | Wrap trong `StringeeUtilProvider` với mock context (personnelId, super_admin) |
| `renderComponent(component, options?)` | Full setup: QueryClient + I18n + Router + StringeeUtilProvider. Trả về `{ user, ...renderResult }` |
| `sleep(timeout)` | Promise-based delay |
| `getRandomInt(min, max)` | Random integer |
| `debugScreen()` | Full DOM debug output |
| `resizeWindow(x, y)` | Simulate window resize |
| `mockImage(url)` | Mock image HTTP response |
| `createRRDMock()` | Mock React Router hooks (useNavigate, useParams) |

## Nguyên tắc Mock (CRITICAL)

**Quy tắc cốt lõi: KHÔNG mock component. Chỉ được mock API hoặc hàm từ thư viện `cogover-comm-web-sdk`.**

Tất cả component con và hàm nội bộ trong component đang test phải **chạy thật** (real render, real logic). Không dùng `vi.mock()` để thay thế component con bằng stub/fake.

### Được phép mock:
- **API calls**: Dùng MSW handlers hoặc mock module API (`src/apis/*`)
- **Hàm từ `cogover-comm-web-sdk`**: Các SDK functions có side effects (socket, WebRTC, v.v.)
- **Browser APIs** không có trong jsdom: `window.open`, `navigator.clipboard`, `IntersectionObserver`, v.v.
- **Translation hooks** (`useT`, `Trans`): Trả về identity function `(key) => key` để test text keys

### KHÔNG được mock:
- Component con trong cùng project (`src/components/*`, `src/pages/*`)
- Hàm helper/util nội bộ (`src/utils/*`, local helpers)
- Hooks nội bộ (`src/hooks/*`, local hooks) — trừ khi hook đó gọi API thì mock ở tầng API
- Component từ `@stringeecom/ui-kit` — render thật
- **`@fortawesome/*` (CRITICAL)**: KHÔNG mock `@fortawesome/react-fontawesome`, `@fortawesome/pro-light-svg-icons`, `@fortawesome/pro-regular-svg-icons`, `@fortawesome/pro-solid-svg-icons`, `@fortawesome/fontawesome-svg-core`. FontAwesome components và icons phải render thật. Nếu test fail do FontAwesome → sửa test setup, KHÔNG mock FontAwesome.

```typescript
// ❌ SAI: Mock component con
vi.mock("src/components/Chat/ChatBox", () => ({
    default: () => <div data-testid="mock-chat-box" />,
}));

// ✅ ĐÚNG: Render thật, mock API nếu cần
import { http, HttpResponse } from "msw";
import { server } from "../../setup";

server.use(
    http.get("/api/conversations", () => {
        return HttpResponse.json({ r: 0, data: [] });
    }, { once: true })
);
renderComponent(<ParentPage />);
```

### Quy trình viết test cho component có component con:

1. **Kiểm tra test đã tồn tại**: Trước khi viết test cho component con, dùng `Glob` kiểm tra trong `src/__tests__/` xem file test tương ứng đã có chưa → tránh duplicate
2. **Viết test bottom-up**: Nếu component con chưa có test → viết test cho component con trước
3. **Viết test component cha**: Render thật toàn bộ tree, assert behavior end-to-end

**Thứ tự viết test trong một folder:**
1. Helpers/utils (pure functions) → không dependency
2. Hooks → mock API nếu cần, KHÔNG mock internal logic
3. Component con (leaf components) → render thật, mock API
4. Component cha → render thật toàn bộ tree (bao gồm component con), mock API

## Yêu cầu Coverage

**Mục tiêu (BẮT BUỘC cả 2 — KHÔNG NEGOTIATE):**
1. **Lines coverage = 100%** cho mỗi file source — KHÔNG được bỏ sót bất kỳ line nào
2. **Branches coverage = 100%** cho mỗi file source — KHÔNG được thiếu bất kỳ nhánh nào

**Tại sao tuyệt đối 100%?**
- Mỗi line / branch chưa cover = một vùng code chưa được verify behavior → có thể ẩn bug mà test không phát hiện.
- "90% là đủ" là cái bẫy: 10% còn lại thường chính là edge case / error path — nơi bug hay nằm nhất.
- Nếu 1 line/branch thực sự "không thể test" → đó là signal code có vấn đề (dead code, defensive thừa) → xoá đi, KHÔNG skip test.

**Quy trình bắt buộc:**
- Ưu tiên cover: happy path → error/edge cases → conditional branches → fallback values → early returns → default params → optional chaining
- Chạy coverage sau mỗi lần viết test → đọc cột `% Lines` và `% Branch` + `Uncovered Line #s`
- Nếu `% Lines < 100` hoặc `% Branch < 100` → **BẮT BUỘC bổ sung test case**, KHÔNG được dừng. Lặp lại tới khi cả 2 cột đều = 100.
- Nếu không thể đạt 100% cho 1 line/branch cụ thể → chỉ 2 lựa chọn:
  1. Xoá line/branch thừa khỏi source (dead code / defensive không cần thiết)
  2. Dùng `/* v8 ignore */` kèm comment lý do defensive rõ ràng (xem "Khi nào được phép < 100% branches" bên dưới)

### 100% Lines + 100% Branches — cover đủ mọi line và mọi nhánh

**Mọi line trong code đều phải được thực thi ít nhất 1 lần trong test.** Mọi điểm rẽ nhánh đều phải có test cover cả 2 phía (hoặc tất cả các nhánh nếu nhiều hơn 2). Không được bỏ qua line/branch vì "khó tạo điều kiện" hay "trường hợp không thể xảy ra" — nếu code có line/branch đó thì test phải cover nó, hoặc xoá khỏi source.

#### Các loại branch phải cover

| Loại | Ví dụ | Cần test |
|------|-------|---------|
| `if / else` | `if (x) { A } else { B }` | 1 case `x` truthy + 1 case falsy |
| Ternary | `cond ? a : b` | 1 case truthy + 1 case falsy |
| Short-circuit `&&` | `a && b()` | `a` truthy (chạy `b()`) + `a` falsy (skip) |
| Short-circuit `\|\|` | `a \|\| b` | `a` truthy + `a` falsy |
| Nullish `??` | `a ?? b` | `a` null/undefined + `a` có giá trị |
| Optional chaining `?.` | `obj?.prop` | `obj` null/undefined + `obj` có giá trị |
| Default param | `function f(x = 10)` | Gọi có `x` + gọi không có `x` |
| `switch / case` | `switch(type)` | 1 case cho mỗi `case` + `default` |
| `try / catch` | `try { a() } catch {}` | 1 case `a()` success + 1 case `a()` throw |
| Early return | `if (!x) return;` | Case trigger return + case không trigger |
| Conditional render | `{show && <Comp />}` | `show=true` + `show=false` |
| Conditional prop | `<X y={a ? 1 : 2} />` | Cả 2 giá trị `a` |

#### Cách phát hiện branch chưa cover

Text reporter của `--coverage.include='src/path/to/file.ts'` có cột **`% Branch`**. Nếu < 100% → có branch chưa cover.

Để biết chính xác branch nào thiếu, dùng `json` reporter rồi parse:

```bash
npm run coverage -- --run src/__test__/path/to/file.test.tsx \
  --coverage.include='src/path/to/file.ts' \
  --coverage.reporter=json-summary \
  --coverage.reporter=json 2>&1 | tail -10
```

Sau đó parse `coverage/coverage-final.json` lấy branch map của file:

```bash
jq '."'"$(pwd)"'/src/path/to/file.ts" | {branchMap, b}' coverage/coverage-final.json | head -100
```

- `branchMap[id]` → vị trí branch trong source (line, type: `if`/`cond-expr`/`switch`/...)
- `b[id]` → array hit count cho từng nhánh (ví dụ `[3, 0]` = nhánh 1 hit 3 lần, nhánh 2 chưa hit → thiếu)

Branch nào có giá trị `0` trong array `b[id]` → đó là branch chưa cover → dựa vào `branchMap[id].loc` để biết line → bổ sung test case.

#### Khi nào được phép < 100% lines / branches

Chỉ 2 trường hợp (áp dụng cho cả lines và branches):
1. **Defensive code không thể trigger**: ví dụ `if (!obj) throw new Error('unreachable')` với `obj` luôn có giá trị theo type. → Giải pháp: xoá defensive branch hoặc dùng `/* v8 ignore next */` comment với lý do rõ ràng.
2. **Branch do TypeScript narrow tự thêm**: hiếm gặp. → Cùng giải pháp `v8 ignore` với comment giải thích.

KHÔNG được dùng `v8 ignore` chỉ vì "lười viết test case" hay "khó mock". Mỗi lần dùng phải có comment lý do cụ thể, reviewer sẽ kiểm tra. Nếu không rõ có phải defensive thật không → viết test cover, KHÔNG ignore.

```typescript
// ✅ OK: có lý do defensive rõ ràng
/* v8 ignore next 3 -- defensive: TypeScript đảm bảo obj không null tại đây */
if (!obj) {
    throw new Error('unreachable');
}

// ❌ SAI: không lý do
/* v8 ignore next */
if (someCondition) { ... }
```

### Workflow tiết kiệm token khi chạy test & coverage

**Quy tắc cốt lõi: KHÔNG bao giờ chạy nguyên suite test/coverage rồi đọc full output. Luôn filter theo đúng 1 file test + 1 file source cần quan tâm, và dùng reporter gọn nhất.**

**KHÔNG dùng `npx` để chạy bất kỳ command test/coverage nào** (không `npx vitest`, không `npx vitest run`, không `npx vitest --coverage`...). Luôn chạy qua script trong `package.json` bằng `npm run test` / `npm run coverage`. Lý do: script đã cấu hình sẵn env, reporter, path alias; `npx` bypass toàn bộ, dễ sai config và làm kết quả không nhất quán giữa các lần chạy.

Output mặc định của vitest/coverage rất dài (hàng trăm file, console.log, stack trace...) → ngốn context cực nhanh. Dùng 3 bước sau:

#### Bước 1: Chạy test cho đúng file → biết pass/fail

```bash
npm run test -- --run src/__test__/path/to/file.test.tsx --reporter=dot 2>&1 | tail -30
```

- `--reporter=dot`: mỗi test chỉ 1 ký tự `.` hoặc `x` → output rất ngắn
- `tail -30`: chỉ lấy phần summary cuối (pass/fail count, duration)
- Nếu có test fail → chạy lại và grep chi tiết lỗi:

```bash
npm run test -- --run src/__test__/path/to/file.test.tsx 2>&1 > /tmp/test.log
grep -nE "FAIL|✗|Error|Expected|Received|AssertionError" /tmp/test.log | head -50
```

#### Bước 2: Chạy coverage CHỈ cho 1 file source (KEY TRICK)

```bash
npm run coverage -- --run src/__test__/path/to/file.test.tsx \
  --coverage.include='src/path/to/file.ts' \
  --coverage.reporter=text \
  --coverage.reporter=json-summary 2>&1 | tail -20
```

- `--coverage.include='src/path/to/file.ts'`: chỉ tính coverage đúng 1 file → text reporter in **duy nhất 1 dòng** bảng kết quả
- Cột `Uncovered Line #s` → **danh sách chính xác line nào chưa được cover**
- `json-summary` reporter đồng thời ghi `coverage/coverage-summary.json` (nhỏ, dễ parse)

Output mẫu (~10 dòng):

```
File          | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
--------------|---------|----------|---------|---------|------------------
 file.ts      |  72.5   |   60.0   |   80.0  |   71.4  | 45-52,78,99-103
```

#### Bước 3: Đọc đúng đoạn code chưa cover → viết thêm test

Dựa vào `Uncovered Line #s`, dùng `Read` với `offset/limit` để đọc đúng vùng code:

```
Read src/path/to/file.ts offset=45 limit=10
Read src/path/to/file.ts offset=99 limit=8
```

Từ đó hiểu branch/logic chưa test → thêm test case → quay lại Bước 1.

#### Khi cần chi tiết hơn (branch coverage)

Nếu text reporter không đủ (ví dụ branch ẩn), đọc nhanh `coverage-summary.json`:

```bash
jq '."src/path/to/file.ts"' coverage/coverage-summary.json
```

→ object nhỏ dạng `{ lines: { pct, total, covered, skipped }, branches: {...}, functions: {...}, statements: {...} }`.

#### Anti-patterns (KHÔNG làm)

- ❌ `npm run test` (chạy nguyên suite) → output hàng nghìn dòng
- ❌ `npm run coverage` không có `--coverage.include` → in bảng của **toàn bộ** file trong project
- ❌ `npx vitest` / `npx vitest run` / `npx vitest --coverage` → bypass script package.json, sai config
- ❌ Đọc file `coverage/coverage-final.json` full → quá lớn, dùng `coverage-summary.json`
- ❌ Đọc full `/tmp/test.log` bằng `Read` không offset → dùng `Grep` hoặc `tail` để lấy phần cần

## Mock Data (BẮT BUỘC — KHÔNG NEGOTIATE)

**Quy tắc cốt lõi (tuyệt đối, không có ngoại lệ): Mọi giá trị mock trong test — bất kể là mock data, mock props, mock response, mock hàm, mock context, mock event... — PHẢI được khai báo đầy đủ theo đúng type của nó. TUYỆT ĐỐI KHÔNG dùng `as unknown as Type`, `as any`, `as Type` để bypass lỗi TypeScript.**

Lý do không đàm phán:
- Mock kiểu `as unknown as Type` che dấu field thiếu → test pass giả, khi type thay đổi runtime mới phát hiện bug.
- User phải review đi review lại để bắt các chỗ lười type. Không chấp nhận — phải đúng ngay lần đầu.
- Nếu type quá phức tạp để viết tay → đó là tín hiệu dùng **factory function** (xem bên dưới) hoặc tách mock data vào `src/__mocks__/`, KHÔNG phải tín hiệu để cast bỏ qua.

### Các hình thức cast bị CẤM

```typescript
// ❌ CẤM tuyệt đối — tất cả các kiểu cast dưới đây đều KHÔNG được dùng trong test
const mockUser = { accountId: '1' } as unknown as GetListDataFieldAccountInfo;
const mockUser = { accountId: '1' } as any as GetListDataFieldAccountInfo;
const mockUser = { accountId: '1' } as GetListDataFieldAccountInfo;
const mockUser = {} as GetListDataFieldAccountInfo;
const mockResponse = fakeData as unknown as ApiResponse<Record>;
const mockProps: ComponentProps = { name: 'x' } as ComponentProps;  // thiếu field bắt buộc
renderComponent(<Comp {...({ partial: true } as Props)} />);
```

Chỉ có 1 cách đúng: khai báo biến với type annotation rồi điền đủ field.

### Cách viết mock data đúng

```typescript
// ❌ SAI: Bỏ qua lỗi TS bằng as unknown
const mockUser = { accountId: '1', fullname: 'User' } as unknown as GetListDataFieldAccountInfo;

// ✅ ĐÚNG: Viết đầy đủ các field theo type
const MOCK_USER_ADMIN: GetListDataFieldAccountInfo = {
    id: 'user-1',
    personnelId: 'personnel-1',
    accountId: 'account-1',
    firstName: 'Admin',
    lastName: 'User',
    fullname: 'Admin User',
    avatar: '',
    email: 'admin@example.com',
    phone: '',
};
```

### Khi type quá lớn → dùng factory function (KHÔNG cast)

Nếu phải viết 30+ field cho 1 object không liên quan tới test case, giải pháp là factory function có default đầy đủ type, nhận `overrides?: Partial<T>`:

```typescript
// ✅ Factory trả về giá trị đầy đủ type, test chỉ override field cần quan tâm
export const createMockRecord = (overrides?: Partial<Record>): Record => ({
    id: 'record-1',
    objectSlug: 'contact',
    createdBy: MOCK_USER_ADMIN,
    created: 0,
    updatedBy: MOCK_USER_ADMIN,
    updated: 0,
    values: {},
    ...overrides,
});

// Dùng trong test — không cast
const record = createMockRecord({ id: 'special-id' });
```

### Tổ chức mock data:

- **Đặt constants vào `src/__mocks__/`** để tái sử dụng giữa nhiều test files
- **Đặt tên constant rõ ràng** để hiểu được dữ liệu mock dùng cho case nào:
  - `MOCK_ROUTING_MANUAL_ASSIGN` — routing với assignType = Manual
  - `MOCK_CHANNEL_FACEBOOK_ACTIVE` — kênh Facebook đang active
  - `SUCCESS_GET_ROUTING_DATA` — response thành công từ API routing
- **Tái sử dụng**: Khi nhiều test files cần cùng mock data → import từ `src/__mocks__/` thay vì copy-paste
- **Factory function** cho mock data có biến thể:

```typescript
// src/__mocks__/sampleMessage.ts
export const createMockSampleMessage = (overrides?: Partial<SampleMessageListResponse>): SampleMessageListResponse => ({
    id: 'sample-1',
    name: 'Test Sample',
    contents: [{ content: 'test', files: [] }],
    createdBy: MOCK_USER_ADMIN,
    created: Date.now(),
    updatedBy: MOCK_USER_ADMIN,
    updated: Date.now(),
    ...overrides,
});
```

## Checklist

- [ ] Import `describe`, `expect`, `it` từ `vitest` (KHÔNG từ jest)
- [ ] Import `vi` từ `vitest` cho mocks (KHÔNG dùng jest.fn())
- [ ] Sử dụng `renderComponent` cho components cần providers
- [ ] Sử dụng `renderHook` với wrapper cho hooks cần providers
- [ ] File `.test.ts` cho pure functions, `.test.tsx` cho hooks/components có JSX
- [ ] Mỗi test case có tên mô tả rõ ràng behavior đang test
- [ ] Test cả happy path và edge cases (null, undefined, empty array, etc.)
- [ ] Không dùng `any` type trong test code
- [ ] KHÔNG mock component con — render thật toàn bộ tree, chỉ mock API và `cogover-comm-web-sdk`
- [ ] KHÔNG mock `@fortawesome/*` — FontAwesome icons và components phải render thật
- [ ] Kiểm tra component con đã có test chưa trước khi viết → tránh duplicate
- [ ] Dùng `userEvent` cho user interaction, KHÔNG dùng `fireEvent`
- [ ] Mock API bằng MSW handler mặc định trong `src/__mocks__/handlers/`, KHÔNG dùng `vi.mock('src/apis/...')`
- [ ] Khi cần nhiều response khác nhau → truyền params khác nhau từ test + if/else trong handler mặc định, HẠN CHẾ `server.use()` + `{ once: true }` trong test body
- [ ] **Mọi giá trị mock** (data, props, response, handler, context, event...) viết đầy đủ theo type — KHÔNG `as unknown as T`, `as any`, `as T`, `{} as T`. Type quá lớn → dùng factory function có `Partial<T>` override
- [ ] Mock data constants đặt trong `src/__mocks__/` để tái sử dụng
- [ ] **Đạt 100% coverage lines cho mỗi file source** — không chấp nhận < 100%. Nếu còn line chưa cover, đọc `Uncovered Line #s` và bổ sung test, hoặc dùng `v8 ignore` kèm lý do defensive rõ ràng
- [ ] **Đạt 100% coverage branches cho mỗi file source** — không chấp nhận < 100%. Nếu < 100%, xác định branch thiếu qua `coverage-final.json` và bổ sung test, hoặc dùng `v8 ignore` kèm lý do rõ ràng
- [ ] Khảo sát cấu trúc thư mục trước khi viết (Bước 0) → viết theo thứ tự leaf-first (Tier 0 → Tier 4)
- [ ] Chạy test filter theo file + `--reporter=dot` + `tail -30` để verify pass (xem "Workflow tiết kiệm token")
- [ ] Chạy coverage với `--coverage.include='src/path/to/file.ts'` để đọc cột `Uncovered Line #s`, KHÔNG chạy coverage toàn project
- [ ] Dùng `npm run test` / `npm run coverage`, KHÔNG dùng `npx vitest` hay bất kỳ command test nào qua `npx`

## Tips

1. **Tách test data ra file riêng** khi có nhiều IO cases → tạo `ioData.ts`
2. **Dùng `vi.fn()`** cho callback props, **`vi.spyOn()`** để spy existing methods
3. **`waitFor`** cho async operations (API calls, state updates)
4. **Mở rộng handler mặc định** theo params thay vì override trong test body. Chỉ dùng `server.use()` + `{ once: true }` cho case cá biệt không thể model bằng params
5. **`server.use()`** chỉ dùng trong test body, KHÔNG dùng trong `beforeAll`
6. **`userEvent.setup()`** trả về từ `renderComponent` → dùng `{ user }` destructuring. Khi dùng `render` thủ công → `const user = userEvent.setup()` trước khi render
7. **KHÔNG mock component hay hàm nội bộ** — chỉ mock API bằng MSW và `cogover-comm-web-sdk`. Render thật toàn bộ component tree
8. **KHÔNG dùng `fireEvent`** — luôn dùng `userEvent` để giả lập hành vi người dùng (click, type, hover, tab, v.v.)
