# Unit Test - @stringeecom/ui-kit

## Tech Stack

- **Test framework**: Vitest (global APIs: `describe`, `it`, `expect`, `vi`)
- **Component testing**: `@testing-library/react` (`render`, `screen`, `waitFor`)
- **User interaction**: `@testing-library/user-event`
- **Hook testing**: `renderHook` from `@testing-library/react`
- **API mocking**: MSW v2 (Mock Service Worker) via `msw/node`
- **Environment**: jsdom, setup file tại `src/__test__/setup.ts`

## Quy trình viết test

### Bước 1: Đọc source file cần test

Đọc toàn bộ file source để hiểu:
- Các functions/hooks/components được export
- Dependencies và imports
- Logic branching, edge cases
- Types và interfaces

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

```typescript
import { http, HttpHandler, HttpResponse } from "msw";
import { _mockApiSuccess } from "../_mock";

// 1. Chuẩn bị mock data
const MOCK_DATA = [{ id: "1", name: "Test Item" }];

// 2. Định nghĩa routes map: service number → handler function
const routes = {
    [myServices.list]: () =>
        HttpResponse.json(_mockApiSuccess({ total: 1, search_after: [], records: MOCK_DATA })),
    [myServices.create]: () =>
        HttpResponse.json(_mockApiSuccess(null)),
} as Record<number, (body: unknown, service: number) => StrictResponse<JsonBodyType>>;

// 3. Router function: đọc header → dispatch đến handler
async function serviceRouter({ request }: ResolverParams) {
    const payload = (await request.json()) as { body: unknown };
    const service = Number(request.headers.get(X_SERVICE_KEY));
    if (!service) return HttpResponse.json({ message: "service not found" }, { status: 404 });
    const fn = routes[service];
    if (!fn) return HttpResponse.json({ message: "route not found" }, { status: 404 });
    return fn(payload, service);
}

// 4. Export MSW handler
export const myHandler: HttpHandler[] = [http.post(myUri.index, serviceRouter)];
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

#### Override handler cho test cụ thể

```typescript
import { http, HttpResponse } from "msw";
import { server } from "../../setup"; // adjust path

it("should handle API error", async () => {
    server.use(
        http.post("/api/endpoint", () => {
            return HttpResponse.json({ r: 1, msg: "Error" });
        }, { once: true })  // ← chỉ chạy 1 lần, sau đó revert về handler mặc định
    );

    renderComponent(<MyComponent />);
    // ... assertions
});
```

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
- **`{ once: true }`** khi override handler trong test body
- **`server.use()`** chỉ dùng trong test body, KHÔNG dùng trong `beforeAll`
- Handler mặc định đặt trong `src/__mocks__/handlers/` — tự động load qua `setup.ts`
- Override chỉ khi cần test case đặc biệt (error, empty data, v.v.)

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

**Mục tiêu: mỗi file test phải đạt >= 90% coverage lines cho file source tương ứng.**

- Ưu tiên cover: happy path → error/edge cases → conditional branches → fallback values
- Nếu chưa đạt 90%, bổ sung test cases cho các nhánh chưa cover (if/else, early return, error handling, edge cases)

### Workflow tiết kiệm token khi chạy test & coverage

**Quy tắc cốt lõi: KHÔNG bao giờ chạy nguyên suite test/coverage rồi đọc full output. Luôn filter theo đúng 1 file test + 1 file source cần quan tâm, và dùng reporter gọn nhất.**

Output mặc định của vitest/coverage rất dài (hàng trăm file, console.log, stack trace...) → ngốn context cực nhanh. Dùng 3 bước sau:

#### Bước 1: Chạy test cho đúng file → biết pass/fail

```bash
npm run test --run src/__test__/path/to/file.test.tsx --reporter=dot 2>&1 | tail -30
```

- `--reporter=dot`: mỗi test chỉ 1 ký tự `.` hoặc `x` → output rất ngắn
- `tail -30`: chỉ lấy phần summary cuối (pass/fail count, duration)
- Nếu có test fail → chạy lại và grep chi tiết lỗi:

```bash
npm run test --run src/__test__/path/to/file.test.tsx 2>&1 > /tmp/test.log
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
- ❌ Đọc file `coverage/coverage-final.json` full → quá lớn, dùng `coverage-summary.json`
- ❌ Đọc full `/tmp/test.log` bằng `Read` không offset → dùng `Grep` hoặc `tail` để lấy phần cần

## Mock Data (BẮT BUỘC)

**Quy tắc cốt lõi: Viết đầy đủ dữ liệu mock theo đúng type, KHÔNG dùng `as unknown as Type` để bỏ qua lỗi TypeScript.**

### Cách viết mock data đúng:

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
- [ ] Mock API bằng MSW handler, KHÔNG dùng `vi.mock('src/apis/...')`
- [ ] Mock data viết đầy đủ theo type, KHÔNG dùng `as unknown as Type`
- [ ] Mock data constants đặt trong `src/__mocks__/` để tái sử dụng
- [ ] Đạt >= 90% coverage lines cho mỗi file source
- [ ] Chạy test filter theo file + `--reporter=dot` + `tail -30` để verify pass (xem "Workflow tiết kiệm token")
- [ ] Chạy coverage với `--coverage.include='src/path/to/file.ts'` để đọc cột `Uncovered Line #s`, KHÔNG chạy coverage toàn project

## Coverage Priority Files

Các file cần bổ sung test để tăng coverage (theo thứ tự ưu tiên):

### Tier 1: Utils (pure functions, dễ nhất)
| File | Current | Uncovered Lines |
|------|---------|----------------|
| `src/utils/app.ts` | 31.4% | ~431 |
| `src/utils/workflow.ts` | 16.0% | ~132 |
| `src/utils/string.ts` | 15.2% | ~104 |
| `src/utils/object.ts` | 17.0% | ~85 |
| `src/utils/cx.ts` | 8.6% | ~76 |
| `src/utils/badge.ts` | 12.9% | ~56 |
| `src/utils/file.ts` | 16.4% | ~55 |
| `src/utils/time.ts` | 54.5% | ~33 |

### Tier 2: Hooks (renderHook)
| File | Current | Uncovered Lines |
|------|---------|----------------|
| `FormBuilder/hooks/useCreateSchema` | 27.9% | ~522 |
| `FormBuilder/hooks/useSubmitFormBuilder` | 13.0% | ~435 |
| `hooks/useSlider` | 1.2% | ~207 |
| `hooks/useFormatCurrency` | 2.1% | ~174 |
| `hooks/useSetRecordValue` | 0.9% | ~148 |
| `hooks/useFilterBuilder` | 0% | ~143 |

### Tier 3: Components (render + userEvent)
| Component | Current | Uncovered Lines |
|-----------|---------|----------------|
| `FormBuilderButtonGroup` | 38.4% | ~713 |
| `DataFields/LongText` | 31.5% | ~710 |
| `RelatedListCards` | 44.7% | ~629 |
| `Table (core)` | 50.5% | ~536 |
| `UploadFile` | 58.7% | ~415 |

## Tips

1. **Tách test data ra file riêng** khi có nhiều IO cases → tạo `ioData.ts`
2. **Dùng `vi.fn()`** cho callback props, **`vi.spyOn()`** để spy existing methods
3. **`waitFor`** cho async operations (API calls, state updates)
4. **`{ once: true }`** khi override MSW handlers để không ảnh hưởng test khác
5. **`server.use()`** chỉ dùng trong test body, KHÔNG dùng trong `beforeAll`
6. **`userEvent.setup()`** trả về từ `renderComponent` → dùng `{ user }` destructuring. Khi dùng `render` thủ công → `const user = userEvent.setup()` trước khi render
7. **KHÔNG mock component hay hàm nội bộ** — chỉ mock API bằng MSW và `cogover-comm-web-sdk`. Render thật toàn bộ component tree
8. **KHÔNG dùng `fireEvent`** — luôn dùng `userEvent` để giả lập hành vi người dùng (click, type, hover, tab, v.v.)
