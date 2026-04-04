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

### Bước 3: Mock API (nếu cần)

#### Override handler cho test cụ thể

```typescript
import { http, HttpResponse } from "msw";
import { server } from "../../setup"; // adjust path

it("should handle API error", async () => {
    server.use(
        http.post("/api/endpoint", () => {
            return HttpResponse.json({ r: 1, msg: "Error" });
        }, { once: true })
    );
    
    renderComponent(<MyComponent />);
    // ... assertions
});
```

#### Sử dụng mock utilities có sẵn

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

#### API Response format chuẩn

```typescript
// Success response
HttpResponse.json({ r: 0, msg: "Success", data: [...], meta: { total: 1 } })

// Error response
HttpResponse.json({ r: 1, msg: "Error message" })

// Record service response (qua recordUri.index)
HttpResponse.json({
    serviceVersion: 1,
    service: 217,
    id: 1000000003,
    type: 1,
    body: {
        data: { search_after: [], total: 1, rows: [...] }
    },
})
```

## File Structure Convention

```
src/__test__/
├── setup.ts                          # MSW server + global setup
├── testUtils.tsx                     # renderComponent, renderWrapper, helpers
├── utils/
│   └── [utilName].test.ts            # Pure function tests
└── components/
    └── [ComponentPath]/
        ├── index.test.tsx            # Component test
        └── hooks/
            └── [hookName]/
                ├── index.test.tsx    # Hook test
                └── ioData.ts         # Test data (nếu nhiều cases)
```

**Quy tắc đặt file:**
- Test file mirror cấu trúc source: `src/components/Foo/Bar.tsx` → `src/__test__/components/Foo/Bar.test.tsx`
- Pure function tests: `src/__test__/utils/[fileName].test.ts`
- Hook tests: `src/__test__/components/[ComponentPath]/hooks/[hookName]/index.test.tsx`
- Component tests: `src/__test__/components/[ComponentPath]/index.test.tsx`

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

## Checklist

- [ ] Import `describe`, `expect`, `it` từ `vitest` (KHÔNG từ jest)
- [ ] Import `vi` từ `vitest` cho mocks (KHÔNG dùng jest.fn())
- [ ] Sử dụng `renderComponent` cho components cần providers
- [ ] Sử dụng `renderHook` với wrapper cho hooks cần providers
- [ ] File `.test.ts` cho pure functions, `.test.tsx` cho hooks/components có JSX
- [ ] Mỗi test case có tên mô tả rõ ràng behavior đang test
- [ ] Test cả happy path và edge cases (null, undefined, empty array, etc.)
- [ ] Không dùng `any` type trong test code
- [ ] Chạy `npx vitest run [path-to-test-file]` để verify test pass
- [ ] Chạy `npx vitest run --coverage [path-to-source-file]` để check coverage tăng

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
6. **`userEvent.setup()`** trả về từ `renderComponent` → dùng `{ user }` destructuring
7. **Không mock internal modules** trừ khi thật sự cần - prefer testing behavior over implementation
