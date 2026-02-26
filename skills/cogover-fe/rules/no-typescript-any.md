---
title: Không Sử Dụng Kiểu `any` Trong TypeScript
impact: CRITICAL
impactDescription: Sử dụng `any` vô hiệu hóa type checking, che giấu bug tại compile-time và làm giảm độ tin cậy của codebase
tags: typescript, any, type-safety, best-practice
---

## Không Sử Dụng Kiểu `any` Trong TypeScript

Không bao giờ sử dụng kiểu `any` trong code TypeScript. Luôn khai báo kiểu cụ thể hoặc sử dụng các kiểu an toàn hơn như `unknown`, `Record<string, unknown>`, generic types, hoặc kiểu union.

Không sử dụng dấu `!` (non-null assertion) để fix lỗi TypeScript. Thay vào đó hãy xử lý `null`/`undefined` một cách tường minh.

### Sai:

```typescript
// ❌ Sử dụng any
const data: any = fetchData()
function handleEvent(event: any) { ... }
const items: any[] = []
const config = {} as any

// ❌ Sử dụng ! để bỏ qua null check
const name = user!.name
const element = document.getElementById('app')!
```

### Đúng:

```typescript
// ✅ Khai báo kiểu cụ thể
const data: UserResponse = fetchData()
function handleEvent(event: MouseEvent) { ... }
const items: Product[] = []

// ✅ Sử dụng unknown khi không biết kiểu
const data: unknown = externalInput
if (typeof data === 'string') {
  // data giờ là string
}

// ✅ Sử dụng Record khi cần object linh hoạt
const config: Record<string, unknown> = {}

// ✅ Sử dụng generic
function parseResponse<T>(response: Response): T { ... }

// ✅ Xử lý null/undefined tường minh
const name = user?.name ?? 'Unknown'
const element = document.getElementById('app')
if (element) {
  // xử lý element
}

// ✅ Sử dụng optional chaining thay vì !
const value = obj?.nested?.property
```

### Tóm tắt:

| Thay vì | Sử dụng |
|---------|---------|
| `any` | Kiểu cụ thể, `unknown`, `Record<string, unknown>`, generic |
| `any[]` | `T[]` với kiểu cụ thể |
| `as any` | Type guard, type assertion đúng kiểu |
| `obj!.prop` | `obj?.prop`, null check, optional chaining |
