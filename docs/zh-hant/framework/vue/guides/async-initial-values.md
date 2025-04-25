---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-25T20:43:43.073Z'
id: async-initial-values
title: 非同步初始值
---
假設您想從 API 獲取一些資料並將其作為表單的初始值。

雖然這個問題表面上看起來很簡單，但背後可能存在您尚未考慮到的隱藏複雜性。

舉例來說，您可能希望在資料獲取期間顯示載入動畫，或是需要優雅地處理錯誤情況。

同樣地，您也可能會尋找快取資料的方法，以避免每次表單渲染時都需要重新獲取。

雖然我們可以從頭實作這些功能，但最終結果可能會與我們維護的另一個專案 [TanStack Query](https://tanstack.com/query) 非常相似。

因此，本指南將展示如何結合使用 TanStack Form 與 TanStack Query 來實現所需行為。

## 基本用法

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
  <p v-if="isLoading">載入中..</p>
  <form v-else @submit.prevent.stop="form.handleSubmit">
    <!-- ... -->
  </form>
</template>
```

這段程式碼會在資料獲取完成前顯示載入文字，之後才會渲染表單並使用獲取的資料作為初始值。
