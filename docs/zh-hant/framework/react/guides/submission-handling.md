---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-25T20:33:27.381Z'
id: submission-handling
title: 提交處理
---

在需要處理多種表單提交類型的情境下，例如一個表單同時包含導向子表單的按鈕與處理標準提交的按鈕，您可以利用 `onSubmitMeta` 屬性與 `handleSubmit` 函式的多載功能來實現。

## 基本用法

首先必須定義 `form.onSubmitMeta` 屬性的預設狀態：

```tsx
const form = useForm({
  defaultValues: {
    firstName: 'Rick',
  },
  // {} 是傳遞給 `onSubmit` 中 `meta` 屬性的預設值
  onSubmitMeta: {} as { lastName: string },
  onSubmit: async ({ value, meta }) => {
    // 透過 handleSubmit 傳遞的值進行操作
    console.log(`${value.firstName} - ${meta}`)
  },
})
```

注意：`onSubmitMeta` 的預設狀態為 `never`，若未提供此屬性卻嘗試在 `handleSubmit` 或 `onSubmit` 中存取時，將會引發錯誤。

接著在呼叫 `onSubmit` 時，可以像這樣傳遞預先定義的 meta 資料：

```tsx
<form
  onSubmit={(e) => {
    e.preventDefault()
    e.stopPropagation()
    form.handleSubmit({
      lastName: 'Astley',
    })
  }}
></form>
```
