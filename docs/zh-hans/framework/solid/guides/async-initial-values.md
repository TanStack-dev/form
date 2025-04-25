---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-12T04:11:00.234Z'
id: async-initial-values
title: 异步初始值
---
假设您需要从 API 获取数据并将其作为表单的初始值使用。

虽然这个问题表面看起来简单，但背后存在一些您可能尚未考虑到的隐藏复杂性。

例如，您可能希望在数据加载时显示加载动画，或者需要优雅地处理错误情况。同样，您可能还需要寻找缓存数据的方法，以避免每次渲染表单时都重新获取数据。

虽然我们可以从头实现这些功能，但最终结果会与我们维护的另一个项目 [TanStack Query](https://tanstack.com/query) 非常相似。

因此，本指南将展示如何结合使用 TanStack Form 和 TanStack Query 来实现所需的行为。

## 基础用法

```tsx
import { createForm } from '@tanstack/solid-form'
import { createQuery } from '@tanstack/solid-query'

export default function App() {
  const { data, isLoading } = createQuery(() => ({
    queryKey: ['data'],
    queryFn: async () => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return { firstName: 'FirstName', lastName: 'LastName' }
    },
  }))

  const form = createForm(() => ({
    defaultValues: {
      firstName: data?.firstName ?? '',
      lastName: data?.lastName ?? '',
    },
    onSubmit: async ({ value }) => {
      // Do something with form data
      console.log(value)
    },
  }))

  if (isLoading) return <p>Loading..</p>

  return null
}
```

这段代码会在数据加载完成前显示加载提示，待数据获取成功后，将使用获取到的数据作为初始值渲染表单。
