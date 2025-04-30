---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:27:40.163Z'
id: async-initial-values
title: 非同期初期値
---

## 非同期初期値

APIからデータを取得し、それをフォームの初期値として使用したい場合を考えます。

この問題は表面上は単純に見えますが、これまで考えていなかった隠れた複雑さが存在します。

例えば、データ取得中にローディングスピナーを表示したい場合や、エラーを適切に処理したい場合があります。同様に、フォームがレンダリングされるたびにデータを取得しなくて済むように、データをキャッシュする方法を探しているかもしれません。

これらの機能の多くを一から実装することも可能ですが、最終的には私たちがメンテナンスしている別のプロジェクト [TanStack Query](https://tanstack.com/query) と非常に似たものになるでしょう。

したがって、このガイドでは、TanStack Form と TanStack Query を組み合わせて目的の動作を実現する方法を紹介します。

## 基本的な使い方

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
  <p>Loading...</p>
{:else}
  <!-- form... -->
{/if}
```

これにより、データが取得されるまでローディングスピナーが表示され、取得されたデータを初期値としてフォームがレンダリングされます。
