---
title: Cách sử dụng hook useMergeRecordFilters trong ui-kit
impact: HIGH
impactDescription: Hook merge nhiều bộ lọc record với logic AND/OR — tự merge sẽ sai logic expression và index numbering
tags: hook, filter, merge, record, logic, AND, OR, CUSTOM, ui-kit
---

## Cách sử dụng hook useMergeRecordFilters trong ui-kit

Đường dẫn hook: `ui-kit/src/hooks/useMergeRecordFilters`

Hook kết hợp nhiều bộ lọc record thành một bộ lọc duy nhất với logic AND/OR tùy chọn.

### Giá trị trả về

| Giá trị | Kiểu | Mô tả |
| --- | --- | --- |
| `mergeRecordFilters` | `(params: MergeRecordParams) => RecordMultipleSelectAdditionalFilters` | Hàm merge filters |

### Params của `mergeRecordFilters`

| Param | Kiểu | Mặc định | Mô tả |
| --- | --- | --- | --- |
| `filters` | `(MergeFilterItem \| undefined)[]` | — | Mảng bộ lọc cần merge |
| `logicType` | `"AND" \| "OR"` | `"AND"` | Kiểu logic kết hợp |

### Interface MergeFilterItem

```tsx
interface MergeFilterItem {
    filterItems: FilterGeneratorItem[];
    logicValue?: string;
    logicType: FilterGeneratorLogicTypes; // "AND" | "OR" | "CUSTOM" | "NONE"
}
```

### RULE-MRF-01: Sử dụng `mergeRecordFilters` thay vì tự merge filterItems — hook xử lý logic expression và index tự động

```tsx
// ❌ Tự merge
const merged = {
    filterItems: [...filter1.filterItems, ...filter2.filterItems],
    logicType: "AND",
};

// ✅ Sử dụng hook — tự động tạo CUSTOM logic với index đúng
const { mergeRecordFilters } = useMergeRecordFilters();
const merged = mergeRecordFilters({
    filters: [filter1, filter2],
    logicType: "AND",
});
// Kết quả: { filterItems: [...], logicType: "CUSTOM", logicValue: "(1 AND 2) AND (3 OR 4)" }
```

### RULE-MRF-02: Filters với `logicType: "NONE"` tự động bị loại bỏ — không cần lọc trước

```tsx
// ✅ Hook tự bỏ qua filter có logicType "NONE"
const merged = mergeRecordFilters({
    filters: [filter1, noneFilter, filter2],
    logicType: "AND",
});
```

### RULE-MRF-03: Truyền `undefined` trong mảng filters được phép — hook tự bỏ qua

```tsx
// ✅ An toàn với undefined
const merged = mergeRecordFilters({
    filters: [filter1, undefined, filter2],
    logicType: "AND",
});
```

### RULE-MRF-04: Kết quả trả về luôn có cấu trúc hợp lệ — khi không có filter nào sẽ trả về object rỗng

```tsx
const merged = mergeRecordFilters({ filters: [] });
// merged = { filterItems: [], logicType: "AND", logicValue: "" }
```

### RULE-MRF-05: Sử dụng `logicType: "OR"` khi cần bất kỳ điều kiện nào thỏa mãn — mặc định là `"AND"`

```tsx
// ✅ Tất cả điều kiện phải thỏa mãn
mergeRecordFilters({ filters: [filter1, filter2], logicType: "AND" });

// ✅ Bất kỳ điều kiện nào thỏa mãn
mergeRecordFilters({ filters: [filter1, filter2], logicType: "OR" });
```
