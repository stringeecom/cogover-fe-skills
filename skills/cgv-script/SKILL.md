---
name: cgv-script
description: Sinh code clientScript cho Layout Rule của form builder Cogover. Dùng khi user mô tả nghiệp vụ tiếng Việt và muốn code paste-ready để dán vào ô "Cấu hình script" của layout. Bắt buộc kích hoạt decision tree 3 câu hỏi khi nghiệp vụ có thao tác SET giá trị (tránh re-render loop + wipe user edit).
---

# cgv-script — sinh code Layout Rule Script

## Mission

User mô tả nghiệp vụ bằng tiếng Việt → output code JavaScript paste-ready cho `pageSettings.script` của layout. Đối tượng user **không biết code** → mọi rủi ro (re-render loop, wipe data, idempotency) phải được skill chặn bằng guard + hỏi tự nhiên.

API runtime: xem `reference/api-cheatsheet.md`.

---

## 🚨🚨🚨 RULE §0 — TUYỆT ĐỐI KHÔNG DÙNG TỪ KHOÁ `return` (CRITICAL)

**Đây là rule QUAN TRỌNG NHẤT của skill. Vi phạm = script lỗi.**

Mọi script Layout Rule được runtime gom vào **CÙNG 1 hàm async duy nhất**. Nếu script này có `return` → các script khác user paste bên dưới (gộp từ nhiều nguồn / nghiệp vụ) sẽ **BỊ CHẶN không chạy**.

### TUYỆT ĐỐI CẤM:

```js
// ❌ SAI — chặn mọi logic phía dưới
if (!customerId) return;
const item = screen.get("address", "FORM_ITEM");
item.value = ...;

// ❌ SAI — early return dù có lý do
if (formType === "view") return;

// ❌ SAI — return trong if-else
if (type === "A") {
    // ...
} else {
    return;
}
```

### LUÔN DÙNG `if (condition) { ... }` block:

```js
// ✅ ĐÚNG — bao bằng if-block, logic phía dưới vẫn chạy
if (customerId) {
    const item = screen.get("address", "FORM_ITEM");
    item.value = ...;
}

// ✅ ĐÚNG — đảo điều kiện
if (formType !== "view") {
    // ... logic chỉ chạy ở create mode
}

// ✅ ĐÚNG — chỉ vào nhánh khi điều kiện thoả
if (type === "A") {
    // ...
} else if (type === "B") {
    // ...
}
// (không có else { return; } — script tự kết thúc)
```

### Vì sao QUAN TRỌNG

User có thể paste **nhiều script từ nhiều nguồn** vào cùng 1 ô "Cấu hình script" của layout (skill này sinh + các script user copy từ task trước + script đồng nghiệp viết). Runtime gom hết → 1 hàm. `return` ở script đầu = các script sau không chạy → user debug khổ.

### Chỉ ngoại lệ duy nhất: `continue` trong loop

```js
// ✅ OK — continue chỉ skip iteration, không thoát script
for (const c of changedRowCells || []) {
    if (c.relatedListSlug !== "order_lines") continue;
    // ...
}
```

### Áp dụng cho TẤT CẢ guard

Mọi guard cũ (`if (formType === "view") return;`, `if (!changedFields["X"]) return;`, `if (rows.length > 0) return;`) **PHẢI** đảo thành `if (condition_đúng) { ... }` block. Xem `safety-rules.md` đã refactor.

---

## 🚨🚨🚨 RULE §1 — TRÁNH VÒNG LẶP VÔ TẬN KHI SET VALUE (CRITICAL)

**Đây là rule QUAN TRỌNG THỨ NHÌ. Vi phạm = browser freeze / crash, mất data, user mất niềm tin.**

### Cơ chế re-trigger (PHẢI NHỚ)

Script Layout Rule **tự re-run** khi 1 trong các điều xảy ra:

1. **Bất kỳ field nào trên form thay đổi value** (`changedFields[slug] = true`)
2. **Bất kỳ cell nào trong related list thay đổi value** (`changedRowCells` populate)
3. **Layout id đổi** (vd qua `screen.changeLayout(...)`)

→ Nghĩa là: **script tự set value rồi tự bị trigger lại bởi chính value đó** = INFINITE LOOP.

### Kịch bản infinite loop điển hình

```js
// ❌ SAI — vòng lặp vô tận
const totalItem = screen.get("total", "FORM_ITEM");
totalItem.value = sum;
// → value đổi → script re-run → set lại → re-run → ... → freeze
```

```js
// ❌ SAI — set cell trong loop không guard
for (const row of rows) {
    const priceCell = row.get("price");
    priceCell.value = computePrice(row);
    // → cell đổi → script re-run → loop tiếp
}
```

### LUÔN DÙNG `if (current !== new) { ... }` TRƯỚC KHI SET

**MỌI** lệnh set `.value` (field hoặc cell) PHẢI có so sánh trước:

```js
// ✅ ĐÚNG — field cha
const totalItem = screen.get("total", "FORM_ITEM");
if (totalItem.value !== sum) {
    totalItem.value = sum;
}

// ✅ ĐÚNG — cell trong related list
const priceCell = row.get("price");
const newPrice = computePrice(row);
if (priceCell.value !== newPrice) {
    priceCell.value = newPrice;
}
```

### Reset value về null cũng phải so sánh

```js
// ❌ SAI — re-set null mỗi lần script chạy (vẫn loop ngầm nếu trigger khác)
applyTypeItem.value = null;

// ✅ ĐÚNG — chỉ reset khi đang có value
if (applyTypeItem.value) {
    applyTypeItem.value = null;
}
```

### Trường hợp KHÔNG cần guard (an toàn)

| API | Trigger re-run? | Cần guard? |
|---|---|---|
| `field.value =` / `cell.value =` | ✅ CÓ | 🚨 **BẮT BUỘC** so sánh |
| `field.limitedOptions =` | ❌ KHÔNG | Optional (so sánh để tránh re-render thừa) |
| `field.display =` | ❌ KHÔNG | Optional |
| `field.readOnly =` | ❌ KHÔNG | Optional |
| `field.required =` | ❌ KHÔNG | Optional |
| `cell.displayHtml =` | ❌ KHÔNG | Optional (cosmetic) |
| `rl.setData(...)` | ⚠️ Có thể | Guard `rows.length === 0` + idempotent payload |

> **Built-in safety net**: runtime có `markParentScriptStart/End` + 200ms debounce cho cell value channel — đỡ 1 phần, nhưng KHÔNG đủ để dựa. Luôn tự guard.

### Multi-field SET → so sánh từng cái

```js
// ✅ ĐÚNG — mỗi field 1 so sánh riêng
if (totalItem.value !== sum) {
    totalItem.value = sum;
}
if (qtyItem.value !== qty) {
    qtyItem.value = qty;
}
```

### Khi SET payload phức tạp (object/array) → so sánh shallow đủ

```js
// ✅ ĐÚNG — array of string IDs, so sánh shallow
const ids = records.map((r) => r.id);
const current = item.limitedOptions || [];
const same = current.length === ids.length && current.every((v, i) => v === ids[i]);
if (!same) {
    item.limitedOptions = ids;
}
```

### Vì sao QUAN TRỌNG

- Browser freeze → user mất data đang nhập.
- React DevTools báo "Maximum update depth exceeded" → user không biết script nào gây ra.
- Có thể không lộ ngay (run 50 lần / giây vẫn smooth ở máy mạnh) → ship production rồi mới phát hiện.
- Debug khó: stack trace chỉ về runtime, không chỉ về script user viết.

→ Phòng từ đầu bằng `if (current !== new)` rẻ hơn debug nhiều lần.

---

## Nguyên tắc tối thượng — CODE PHẢI DỄ HIỂU KHI MỚI ĐỌC

User là **non-coder**, sau này sẽ phải tự đọc lại / sửa script. Mọi dòng code phải readable ngay từ lần đọc đầu tiên — KHÔNG được trade clarity lấy "elegance" hay "smart code".

### Các quy tắc cụ thể

1. **Tên biến nói tiếng người**, không viết tắt tối nghĩa:
   - ✅ `const orderId = $record.order;`
   - ❌ `const oid = $record.order;` / `const x = $record.order;`

2. **Mỗi dòng làm 1 việc**, không gom nhiều bước thành 1 expression dài:
   - ✅ ```js
     const { records } = await filterRecords("customer", { filterItems: [...] });
     const customer = records[0];
     const address = customer?.defaultAddress ?? "";
     ```
   - ❌ ```js
     const address = (await filterRecords("customer", { filterItems: [...] })).records[0]?.defaultAddress ?? "";
     ```

3. **Tránh ternary lồng nhau / optional chaining sâu**. Chia thành if-block rõ ràng:
   - ✅ ```js
     if (customerId) {
         const item = screen.get("address", "FORM_ITEM");
         if (!item.value) {  // chưa có địa chỉ → mới fill
             item.value = ...;
         }
     }
     ```
   - ❌ ```js
     screen.get("address", "FORM_ITEM").value ||
       (customerId && (screen.get("address", "FORM_ITEM").value = await ...));
     ```

4. **KHÔNG dùng functional combinators (`reduce`/`flatMap`/curry/pipe)** khi `for...of` đơn giản đủ dùng. User đọc `for (const row of rows) { ... }` dễ hơn `rows.reduce((acc, row) => ...)`.

5. **KHÔNG abstract sớm**. 1 script chỉ chạy 1 nghiệp vụ — đừng tạo helper function `applyOverride()`, `withGuard()`, factory… trừ khi có **lý do bắt buộc** (vd dùng lại 3+ lần trong cùng script).

6. **Comment giải thích WHY khi guard / pass-through không hiển nhiên**:
   - ✅ `if (existingRows.length === 0) { /* bảng còn rỗng → mới seed, tránh ghi đè data user */ }`
   - ✅ `rl.setData(records);  // records từ filterRecords đã ở API shape, pass thẳng — converter xử`
   - ❌ Comment giải thích cái code đã tự nói: `// set value vào field` (thừa)

7. **Comment header `⚠️`** đầu script là BẮT BUỘC khi có set value — liệt kê 3 dòng: chạy khi nào / KHÔNG chạy khi / KHÔNG ghi đè. Đây là thứ user xem trước tiên khi mở lại script.

8. **Hardcode rõ ràng > config thông minh**. Slug field, slug bảng, threshold → viết thẳng làm string literal, đừng tạo `const CONFIG = { ... }` cho 1-2 hằng.

9. **Đừng "phòng thủ" cho case không tồn tại**. Vd nếu đã `if (!orderId) return;` rồi thì không cần `customerId?.toString() ?? ""` ở dòng tiếp.

10. **Output ngắn tốt hơn output đầy đủ-nhưng-rườm-rà**. Nếu cùng nghiệp vụ làm được trong 15 dòng → đừng viết 40 dòng.

> **Test "dễ đọc"**: trước khi output, đọc lại script với mindset của user non-coder. Nếu có 1 dòng phải đọc lần 2 mới hiểu → viết lại.

---

## Workflow (làm theo đúng thứ tự)

### Bước 1 — Đọc API cheatsheet
Luôn `Read` file `reference/api-cheatsheet.md` đầu tiên. Đây là knowledge base bắt buộc — không sinh code mà chưa đọc.

### Bước 2 — Parse mô tả của user
Trích 4 entity:
- **Target**: field/cell/related-list nào (slug nếu user nói; nếu không nói → placeholder + nhắc trong output).
- **Trigger**: khi nào script chạy (mount / changedFields / changedRowCells).
- **Action**: set value / show-hide / readOnly / required / submit / displayHtml / fetch.
- **Pattern**: seed / reactive / one-shot / aggregate / validate.

### Bước 2.5 — Hỏi technical info còn thiếu (BẮT BUỘC)

Code script cần đúng **slug** và **fieldType** kỹ thuật. Nếu user mô tả nghiệp vụ chung (vd "khi đổi khách hàng thì fill địa chỉ") mà chưa cho biết slug/type → **bắt buộc** hỏi trước khi đi tiếp.

Các info phải biết để sinh code đúng:

| Info | Khi nào cần | Ví dụ |
|---|---|---|
| **Object slug** (đang config layout) | Luôn — để biết script chạy trên object nào | `return_order`, `order`, `customer` |
| **Field slug** + **fieldType** | Mỗi field nhắc trong mô tả (target SET, trigger field, field đọc data) | `customer` (lookup), `address` (short_text), `quantity` (number), `delivery_date` (date) |
| **Related list slug TRONG LAYOUT** | Khi nghiệp vụ đụng tới bảng (related list) | ⚠️ Xem cảnh báo dưới |

#### ⚠️ Cảnh báo CỰC QUAN TRỌNG về related list slug

**Slug của related list trong layout KHÁC slug của object liên kết.**

- Khi designer kéo related list vào layout → đặt 1 slug riêng cho related list đó **trong phạm vi layout này**.
- Vd: layout `return_order` có related list trỏ tới object `order_line`, designer đặt slug bảng đó là `return_order_lines` (HOẶC `lines`, HOẶC bất kỳ tên gì designer chọn).
- **Trong script PHẢI dùng slug của related list trong layout**, không phải slug của object đích (`order_line`).
- Sai slug → `screen.get("order_line", "RELATED_LIST")` trả `null` → script silent fail.

Khi hỏi user, **luôn** nhấn mạnh:
> "Slug của related list **trong layout** là gì? Đây là slug designer đặt khi kéo bảng vào layout. Vào trang Sửa giao diện → click vào bảng → check field 'slug'. KHÔNG phải slug của object kia."

#### Pattern hỏi (gộp 1 lần nếu thiếu nhiều info)

Dùng `AskUserQuestion` với 1-3 câu hỏi (mỗi câu cover 1 cluster info), HOẶC hỏi text trực tiếp với checklist:

```
Để sinh code chính xác, cho mình biết các slug và type:

1. **Object** đang config layout: slug là gì? (vd: `return_order`)
2. **Field "khách hàng"**: slug? fieldType? (vd: slug=`customer`, type=`lookup`)
3. **Field "địa chỉ"**: slug? fieldType? (vd: slug=`address`, type=`short_text`)
4. **Bảng "chi tiết đơn"**: slug **TRONG LAYOUT** là gì? ⚠️ KHÔNG phải slug của object kia.
   (Vào Sửa giao diện → click vào bảng → xem field 'slug')
```

Nếu user trả lời thiếu → hỏi lại đúng item còn thiếu. KHÔNG đoán slug. KHÔNG dùng placeholder `/* SLUG */` nữa — phải có slug thực tế hoặc dừng workflow.

### Bước 3 — Detect DANGER MODE (BẮT BUỘC)

Mô tả có 1 trong các keyword sau → đi vào **DANGER MODE**:
- "set", "gán", "đổi giá trị", "fill", "tự động điền"
- "lấy từ X đổ vào Y", "copy", "sao chép", "lấy data"
- "seed", "tự động lưu", "submit"
- "khi chọn X thì set/đổi/fill Y"

→ Mở `reference/safety-rules.md` + đi qua **Decision tree** bên dưới. **Không skip dù mô tả có vẻ đầy đủ.**

Mô tả KHÔNG có set value (chỉ show/hide/readOnly/highlight) → skip decision tree, đi thẳng Bước 5.

### Bước 4 — Decision tree (chỉ khi DANGER MODE)

Dùng `AskUserQuestion` lần lượt 3 câu sau. Format option phải có **ví dụ cụ thể tiếng Việt**, không dùng thuật ngữ kỹ thuật:

#### Q1 — Khi nào chạy?
Header: "Khi nào chạy"
Question: "Script này nên chạy khi nào?"
Options:
1. "Khi user vừa mở form" — Ví dụ: cứ mở form ra là script chạy, dù form mới hay form đã lưu.
2. "Khi user mới tạo bản ghi" — Ví dụ: chỉ chạy ở trang Tạo mới, KHÔNG chạy khi vào form đã lưu để xem/sửa.
3. "Khi user đổi 1 ô cụ thể" — Ví dụ: khi user đổi ô 'Khách hàng' → tự fill địa chỉ. Sẽ hỏi tên ô ở câu sau.
4. "Khi user gõ vào ô trong bảng" — Ví dụ: khi user gõ vào ô 'Tên SP' trong dòng → tự fill ô 'Giá' cùng dòng.

#### Q2 — Re-run on edit? (CHỈ hỏi nếu Q1 = option 1)
Header: "Mở form đã lưu"
Question: "Khi user mở lại form đã lưu trước đó để xem/sửa, script có cần chạy lại không?"
Options:
1. "KHÔNG cần chạy lại (an toàn nhất)" — Ví dụ: form Return Order đã lưu, user mở ra xem → KHÔNG tự seed lại order_lines, giữ nguyên data đã save.
2. "Có cần chạy lại" — Cảnh báo: chỉ chọn khi chắc chắn muốn ghi đè, có thể xoá thay đổi user đã làm.

#### Q3 — Overwrite policy?
Header: "Ghi đè data"
Question: "Trước khi script đổi giá trị, target (ô hoặc bảng) có thể đã có sẵn data user nhập không?"
Options:
1. "Có thể có — KHÔNG được ghi đè" — Ví dụ: bảng order_lines có thể có dòng user tự thêm, script không được xoá.
2. "Chắc chắn rỗng — overwrite được" — Ví dụ: field tự động luôn rỗng lúc đầu, fill cái mới vào không vấn đề.
3. "Luôn overwrite" — Cảnh báo: sẽ xoá data user nhập tay nếu có. Chỉ chọn khi business yêu cầu reset.

#### Q4 (follow-up tuỳ) — Tên trigger field/cell (CHỈ khi Q1 = 3 hoặc 4 và user chưa cho biết slug)
Hỏi text trực tiếp (không AskUserQuestion):
- Q1=3 → "Tên slug của field user đổi là gì?" (vd `order`, `customer`)
- Q1=4 → "Tên slug bảng + slug ô là gì?" (vd bảng `order_lines`, ô `product`)

### Bước 5 — Map answers → recipe + guard variant

Đọc `reference/recipes.md` để chọn recipe. Áp guard theo bảng (từ `safety-rules.md`).

> ⚠️ **NHẮC LẠI RULE §0**: TUYỆT ĐỐI KHÔNG dùng `return`. Mọi guard dưới đây dùng `if (positive_condition) { ... }` block bao quanh logic, KHÔNG được dùng `if (negative_condition) return;`.

| Q1 | Q2 | Q3 | Guard wrapper (if-block) |
|---|---|---|---|
| 1 mọi lần mở | 1 không re-seed | 1 không overwrite | `if (formType !== "view") { if (rows.length === 0) { ... } }` + (optional) `$ref.seeded` |
| 1 mọi lần mở | 1 không re-seed | 2 chắc chắn rỗng | `if (formType !== "view") { ... }` |
| 1 mọi lần mở | 2 có re-seed | 3 luôn overwrite | banner ⚠️ — chạy thẳng, không cần guard |
| 2 chỉ tạo mới | n/a | 1 không overwrite | `if (formType !== "view") { if (rows.length === 0) { ... } }` |
| 2 chỉ tạo mới | n/a | 2 chắc chắn rỗng | `if (formType !== "view") { ... }` |
| 3 user đổi field X | n/a | 1 không overwrite | `if (changedFields["X"]) { if (!target.value) { ... } }` |
| 3 user đổi field X | n/a | 2 chắc chắn rỗng | `if (changedFields["X"]) { ... }` |
| 3 user đổi field X | n/a | 3 luôn overwrite | `if (changedFields["X"]) { ... }` + ⚠️ banner |
| 4 user gõ cell | n/a | bất kỳ | `for (const c of changedRowCells \|\| []) { if (c.relatedListSlug === "X" && c.fieldSlug === "Y") { ... } }` — KHÔNG dùng `continue` trừ khi loop body có nhiều bước |

### Bước 6 — Sinh code với guard + so sánh trước khi set

> ⚠️ **NHẮC LẠI RULE §0**: TUYỆT ĐỐI KHÔNG dùng `return`. Bao toàn bộ logic bằng `if (positive_condition) { ... }` block.

**Mọi set value field cha** bắt buộc kèm so sánh:
```js
const item = screen.get("X", "FORM_ITEM");
if (item.value !== newValue) {
    item.value = newValue;  // tránh re-render loop
}
```

**Mọi rl.setData(...)** bắt buộc check `existingRows.length === 0` (bằng if-block) trừ khi Q3=3.

**Header comment tự động** (auto-prepend khi có DANGER MODE):
```js
/**
 * ⚠️ SCRIPT NÀY THAY ĐỔI DỮ LIỆU FORM TỰ ĐỘNG
 *
 * Chạy khi: <Q1 answer>
 * KHÔNG chạy khi: <Q2 logic>
 * KHÔNG ghi đè: <Q3 logic>
 *
 * Nếu sửa script này, GIỮ NGUYÊN các điều kiện `if (...)` bao quanh logic.
 * TUYỆT ĐỐI KHÔNG thêm `return` — script này nằm chung 1 hàm với
 * các script khác, `return` sẽ chặn chúng không chạy.
 */
```

### Bước 7 — Verify checklist trước khi output

Tự kiểm 10 điểm:

0. 🚨 **TUYỆT ĐỐI KHÔNG có từ khoá `return`** ở bất kỳ đâu trong script (kể cả trong if-else, kể cả "có lý do chính đáng"). Script chạy chung 1 hàm với scripts khác — `return` chặn hết. Grep code vừa sinh xem có `return` không. CÓ → fix bằng cách bao if-block. Ngoại lệ duy nhất: `continue` trong `for` loop.
1. 🚨 **MỌI SET `.value` (field hoặc cell) PHẢI có `if (current !== new) { current = new; }`** — RULE §1. Re-trigger script là tự động → set value không guard = infinite loop. Grep code: tìm mọi dòng có `.value =` (cả `field.value`, `cell.value`, `item.value`, `row.get(...).value`) → check ngay phía trên có `if (... !== ...)` không. Set `null` cũng phải so sánh `if (item.value) { item.value = null; }`.
2. Mọi `rl.row/rows/findRow/submit/filterRecords` có `await` chưa?
3. Value SET cho `row.get(slug).value` / `field.value` có khớp **§7 SCRIPT shape** không? (date → `Date`, time → ms, range → `{ gte, lte }`, lookup → **string ID** — KHÔNG phải `{ id, ... }`)
4. `rl.setData(records)` có dùng **§7B API record shape** không? (date → **epoch ms number** — KHÔNG `Date` object; lookup → chấp `RecordItem | string | array`). KHÔNG nhầm 2 shape.
5. SET bảng có guard `if (rows.length === 0) { rl.setData(...) }` (trừ khi user chọn overwrite)? Payload setData có idempotent (cùng input → cùng output) không?
6. Guard wrapper theo bảng Bước 5 đã có? (if-block, KHÔNG phải `if (...) return;`)
7. **KHÔNG có self-cache** kiểu `if ($ref.xxx) ...; / $ref.cached = await filterRecords(...)`. `filterRecords` đã có React Query cache built-in — wrap thừa = code rác + data stale.
8. **Code dễ đọc khi mới nhìn** (xem "Nguyên tắc tối thượng" đầu file): tên biến nói tiếng người, mỗi dòng 1 việc, không ternary lồng, không helper function thừa, không abstract sớm, không "phòng thủ" cho case không tồn tại. Đọc lại script với mindset user non-coder — nếu phải đọc 1 dòng tới lần 2 mới hiểu → viết lại.
9. **`changeLayout` / `triggerButton`** có guard chống loop? (vd `if ($ref.lastLayout !== id) { ... }` cho changeLayout)

Lỗi → fix in-place. Không output code chưa pass 10/10. **Điểm 0 hoặc điểm 1 sai = REJECT toàn bộ code.**

### Bước 8 — Output

Format BẮT BUỘC (4 phần):

```markdown
## Code

[code block JavaScript đầy đủ — paste vào ô Cấu hình script]

## Script này làm gì

[3-5 dòng tiếng Việt thân thiện: chạy khi nào, kiểm tra gì, đổi gì, có gì để bảo vệ data user]

## 🧪 Bắt buộc test trước khi đưa lên production

1. [test case 1 cụ thể với business]
2. [test case 2]
3. [test case 3]
4. [test case 4 — chú ý edge case Q2/Q3 user đã chọn]

## 📖 Thuật ngữ trong code

[Glossary cá nhân hoá: CHỈ in những thuật ngữ thực sự xuất hiện trong code lần này. Tra từ Glossary Dictionary bên dưới.]
```

---

## Glossary Dictionary (để skill tra cứu khi build output)

Khi output, scan code vừa sinh, **chỉ in** entries match những từ xuất hiện:

| Trong code | Giải thích tiếng Việt |
|---|---|
| `changedFields["X"]` | "user vừa đổi field tên X" (true/false) |
| `changedRowCells` | "danh sách các ô bảng user vừa gõ" (mảng) |
| `await` | "đợi" — đứng trước hàm cần đợi xong mới chạy tiếp |
| `$record` | "bản ghi đang sửa" — chứa giá trị các field |
| `$record.id` | "ID của bản ghi" — nếu có nghĩa là form đã lưu rồi |
| `formType` | "loại form đang mở" — `"create"` = Tạo mới, `"view"` = đã lưu (xem/sửa) |
| `$parentRecord` | "bản ghi cha" — khi form con (modal/related list) |
| `$currentPersonnel` | "nhân viên đang đăng nhập" |
| `$ref` | "bộ nhớ tạm của script" — giữ qua các lần script chạy lại trong 1 session |
| `screen.get(slug, "FORM_ITEM")` | "lấy field có slug = X" |
| `screen.get(slug, "RELATED_LIST")` | "lấy bảng (related list) có slug = X" |
| `screen.changeLayout(id)` | "chuyển sang layout khác" |
| `screen.triggerButton(slug)` | "tự động bấm button có slug = X" |
| `field.value` | "giá trị của field" — đọc và ghi được |
| `field.display = false` | "ẩn field đi, không hiển thị" |
| `field.readOnly = true` | "khoá field, không cho sửa" |
| `field.required = true` | "bắt buộc nhập field này" |
| `field.limitedOptions` | "giới hạn option chọn (cho select)" |
| `rl.setData([...])` | "ghi đè toàn bộ dòng trong bảng bằng mảng record mới (theo API record shape — xem §7B cheatsheet, KHÁC SCRIPT shape của `row.get().value`)" |
| `rl.rows()` | "lấy danh sách dòng hiện có trong bảng" |
| `rl.row(i)` | "lấy dòng thứ i (0 là dòng đầu)" |
| `rl.findRow(predicate)` | "tìm 1 dòng theo điều kiện" |
| `rl.submit()` | "tự động lưu bảng — không cần user bấm nút Lưu" |
| `row.get(slug)` | "lấy ô có slug = X trong dòng" |
| `cell.displayHtml` | "đổi cách hiển thị ô (chỉ visual, không lưu data)" |
| `filterRecords(slug, params)` | "query danh sách bản ghi từ object khác" |
| `isDirtyForm` | "form có thay đổi chưa lưu hay không" |
| `new Date()` | "ngày giờ hiện tại" |

---

## Reference files

| File | Khi nào đọc |
|---|---|
| `reference/api-cheatsheet.md` | LUÔN — Bước 1 |
| `reference/safety-rules.md` | Khi vào DANGER MODE — Bước 3+5 |
| `reference/recipes.md` | Bước 5 — chọn template |
| `reference/case-studies.md` | Khi user mô tả khớp 1 case study đã có (vd "set order_line từ order") — đọc để clone pattern |

---

## Anti-patterns (CẤM)

0. 🚨 **DÙNG `return` TRONG SCRIPT** — CỰC KỲ NGHIÊM TRỌNG. Script Layout Rule chạy chung 1 hàm async với các script khác user paste vào. `return` ở script này → chặn mọi logic phía dưới (kể cả script đồng nghiệp viết, script copy từ task khác). LUÔN dùng `if (positive_condition) { ... }` block. Ngoại lệ duy nhất: `continue` trong `for` loop. Xem RULE §0 đầu file.
0.5. 🚨 **SET `.value` không có `if (current !== new)` guard** — TẠO VÒNG LẶP VÔ TẬN. Re-trigger script là cơ chế built-in: mỗi lần value field/cell thay đổi → script chạy lại. Set value mà không so sánh → infinite loop → browser freeze. Áp dụng cho mọi `field.value =`, `item.value =`, `row.get(...).value =`, `cell.value =`. Xem RULE §1 đầu file.
1. **Sinh code khi chưa biết đầy đủ slug + fieldType + related list slug**. Đoán = sai. Luôn hỏi.
2. **Nhầm slug related list trong layout với slug object đích**. Phải nhắc user "slug trong layout, không phải object kia" mỗi lần hỏi.
3. **Sinh code mà chưa hỏi Q1/Q2/Q3** khi nghiệp vụ có set value. Dù user nói chi tiết, vẫn phải hỏi để confirm intent.
4. **Bỏ comment header `⚠️`** khi script có set value. User maintain sau sẽ không biết guard tại sao có.
5. **Output code thiếu test checklist**. User cần biết phải test gì.
6. **Set value field cha không kèm `if (item.value !== ...)`**. Trigger re-render loop.
7. **rl.setData không kèm `rows.length === 0`** trừ khi user chọn overwrite explicit.
8. **Dùng thuật ngữ kỹ thuật trong câu hỏi**. User non-coder. Luôn nói "khi user mở form / khi user đổi ô" thay vì "trigger", "mount", "changedFields".
9. **Sinh > 1 script trong cùng output**. 1 turn = 1 script. Nếu user mô tả 2 nghiệp vụ → hỏi tách hoặc gộp.
10. **Tự cache kết quả `filterRecords` vào `$ref`** (vd `if ($ref.products) return;`, `$ref.cached = await filterRecords(...)`). `filterRecords` đã có cache built-in của React Query — gọi lại cùng params = cache hit. Self-cache = code thừa + data stale. `$ref` chỉ dùng cho state script thật (`seeded`, `submitted`, counter, last-layout-id chống loop swap).
11. **Nhầm shape `rl.setData(records)` với SCRIPT shape của `row.get().value`**. `setData` nhận **API record shape** (§7B): `date` = epoch ms number, `lookup` = `RecordItem | string | array`. `row.get().value` nhận **SCRIPT shape** (§7): `date` = `Date`, `lookup` = string ID. Đừng truyền `new Date(...)` cho field date trong setData → blank. Mặc định: nếu records lấy từ `filterRecords` → pass thẳng, KHÔNG map thủ công.
12. **Code "thông minh" khó đọc** — gom nhiều bước vào 1 expression dài, ternary lồng nhau, optional chaining sâu 4+ tầng, dùng `reduce`/`flatMap`/curry khi `for...of` đủ dùng, tạo helper function chỉ dùng 1 lần, abstract sớm (factory, hooks-style trong script), tên biến viết tắt tối nghĩa (`oid`, `c`, `x`). User non-coder sẽ không hiểu, không sửa được, hỏng business. Xem "Nguyên tắc tối thượng" đầu file.
