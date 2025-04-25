---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-12T04:09:29.672Z'
id: async-initial-values
title: 异步初始值
---
假设您需要从 API 获取数据并将其作为表单的初始值。

虽然这个问题表面看起来简单，但背后隐藏着您可能尚未考虑的复杂性。

例如，您可能希望在数据加载时显示加载动画，或者需要优雅地处理错误情况。

同样，您可能还需要寻找缓存数据的方法，以避免每次渲染表单时都重新获取数据。

虽然我们可以从头实现这些功能，但最终方案会与我们维护的另一个项目 [TanStack Query](https://tanstack.com/query) 非常相似。

因此，本指南将展示如何结合使用 TanStack Form 和 TanStack Query 来实现所需行为。

## 基础用法

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'
import { useQuery } from '@tanstack/vue-query'
import { watchEffect, reactive } from 'vue'

const { data, isLoading } = useQuery({
  queryKey: ['data'],
  queryFn: async () => {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return { firstName: 'FirstName', lastName: 'LastName' }
  },
})

const firstName = computed(() => data.value?.firstName || '')
const lastName = computed(() => data.value?.lastName || '')

const defaultValues = reactive({
  firstName,
  lastName,
})

const form = useForm({
  defaultValues,
  onSubmit: async ({ value }) => {
    // Do something with form data
    console.log(value)
  },
})
</script>

<template>
  <p v-if="isLoading">加载中..</p>
  <form v-else @submit.prevent.stop="form.handleSubmit">
    <!-- ... -->
  </form>
</template>
```

这段代码会在数据加载时显示"加载中"文字，待数据获取完成后，将使用获取到的数据作为初始值渲染表单。
