---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:25:43.641Z'
id: async-initial-values
title: 非同期初期値
---

APIからデータを取得し、それをフォームの初期値として使用したい場合を考えてみましょう。

一見単純な問題に思えますが、これまで気づかなかった複雑な要素が潜んでいる可能性があります。

例えば、データ取得中にローディングスピナーを表示したい場合や、エラーを適切に処理したい場合が考えられます。

同様に、フォームがレンダリングされるたびにデータを取得しなくて済むように、データをキャッシュする方法を探しているかもしれません。

これらの機能の多くを一から実装することも可能ですが、最終的には私たちがメンテナンスしている別のプロジェクト [TanStack Query](https://tanstack.com/query) と非常に似たものになるでしょう。

したがって、このガイドでは、TanStack Form と TanStack Query を組み合わせて目的の動作を実現する方法を紹介します。

## 基本的な使い方

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
  <p v-if="isLoading">Loading..</p>
  <form v-else @submit.prevent.stop="form.handleSubmit">
    <!-- ... -->
  </form>
</template>
```

このコードは、データが取得されるまで「Loading..」というテキストを表示し、データが取得されるとそのデータを初期値としてフォームをレンダリングします。
