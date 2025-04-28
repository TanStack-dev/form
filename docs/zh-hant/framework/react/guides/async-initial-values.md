---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-25T20:33:29.078Z'
id: async-initial-values
title: 非同步初始值
---

假設您想從 API 獲取一些資料，並將其作為表單的初始值使用。

雖然這個問題表面上看起來很簡單，但背後可能隱藏著您尚未考慮到的複雜性。

舉例來說，您可能希望在資料獲取期間顯示載入動畫，或是優雅地處理錯誤情況。同樣地，您也可能會尋找快取資料的方法，以避免每次渲染表單時都需要重新獲取資料。

雖然我們可以從頭實作這些功能，但最終結果可能會與我們維護的另一個專案 [TanStack Query](https://tanstack.com/query) 非常相似。

因此，本指南將向您展示如何結合使用 TanStack Form 與 TanStack Query 來實現所需行為。

## 基本用法

```tsx
import { useForm } from '@tanstack/react-form'
import { useQuery } from '@tanstack/react-query'

export default function App() {
  const {data, isLoading} = useQuery({
    queryKey: ['data'],
    queryFn: async () => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return {firstName: 'FirstName', lastName: "LastName"}
    }
  })

  const form = useForm({
    defaultValues: {
      firstName: data?.firstName ?? '',
      lastName: data?.lastName ?? '',
    },
    onSubmit: async ({ value }) => {
      // Do something with form data
      console.log(value)
    },
  })

  if (isLoading) return <p>Loading..</p>

  return (
    // ...
  )
}
```

這段程式碼會在資料獲取完成前顯示載入提示，然後使用獲取到的資料作為初始值來渲染表單。
