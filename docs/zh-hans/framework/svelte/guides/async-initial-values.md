---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-29T23:14:38.522Z'
id: async-initial-values
title: 异步初始值
---

假设您需要从 API 获取数据并将其作为表单的初始值使用。

虽然这个问题表面看起来简单，但其中可能隐藏着您尚未考虑到的复杂性。

例如，您可能希望在数据加载时显示加载动画，或者需要优雅地处理错误情况。同样，您可能还需要寻找缓存数据的方法，以避免每次渲染表单时都重新获取数据。

虽然我们可以从头实现这些功能，但最终方案会与我们维护的另一个项目 [TanStack Query](https://tanstack.com/query) 非常相似。

因此，本指南将展示如何结合使用 TanStack Form 和 TanStack Query 来实现所需的行为。

## 基础用法

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
        // 对表单数据进行操作
        console.log(value)
      },
    }))
</script>

{#if $isLoading}
  <p>加载中...</p>
{:else}
  <!-- 表单内容... -->
{/if}
```

这段代码会在数据加载时显示"加载中..."提示，待数据获取完成后，将使用获取到的数据作为初始值渲染表单。
