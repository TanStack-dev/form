---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-25T20:47:47.031Z'
id: async-initial-values
title: 非同步初始值
---
假設您想從 API 取得一些資料，並將其作為表單的初始值使用。

雖然這個問題表面上看起來很簡單，但背後可能存在一些您尚未考慮到的複雜性。

舉例來說，您可能希望在資料載入時顯示載入動畫，或是需要優雅地處理錯誤情況。同樣地，您可能也會想尋找快取資料的方法，以避免每次渲染表單時都需要重新取得資料。

雖然我們可以從頭實作這些功能，但最終結果可能會與我們維護的另一個專案 [TanStack Query](https://tanstack.com/query) 非常相似。

因此，本指南將展示如何結合使用 TanStack Form 與 TanStack Query 來達成預期的行為。

## 基本用法

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'
  import { createQuery } from '@tanstack/svelte-query'

    const { data, isLoading } = createQuery(() => ({
      queryKey: ['data'],
      queryFn: async () => {
        await new Promise((resolve) => setTimeout(resolve, 1000))
        return { firstName: 'FirstName', lastName: 'LastName' }
      },
    }))

    const form = createForm(() => ({
      defaultValues: {
        firstName: $data?.firstName ?? '',
        lastName: $data?.lastName ?? '',
      },
      onSubmit: async ({ value }) => {
        // Do something with form data
        console.log(value)
      },
    }))
</script>

{#if $isLoading}
  <p>載入中...</p>
{:else}
  <!-- form... -->
{/if}
```

這段程式碼會在資料載入完成前顯示載入提示，待資料取得後，便會以取得的資料作為初始值來渲染表單。
