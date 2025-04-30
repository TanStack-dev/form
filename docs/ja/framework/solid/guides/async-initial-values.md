---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:27:07.405Z'
id: async-initial-values
title: 非同期初期値
---

APIからデータを取得し、それをフォームの初期値として使用したい場合を考えてみましょう。

この問題は表面上は単純に見えますが、これまで考えもしなかったような複雑さが潜んでいます。

例えば、データ取得中にローディングスピナーを表示したい場合や、エラーを適切に処理したい場合があります。
同様に、フォームがレンダリングされるたびにデータを取得しなくて済むように、データをキャッシュする方法を探しているかもしれません。

これらの機能の多くを一から実装することも可能ですが、その結果は私たちがメンテナンスしている別のプロジェクト [TanStack Query](https://tanstack.com/query) と非常に似たものになるでしょう。

したがって、このガイドでは、TanStack Form と TanStack Query を組み合わせて目的の動作を実現する方法を紹介します。

## 基本的な使い方

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
      // フォームデータで何か処理を行う
      console.log(value)
    },
  }))

  if (isLoading) return <p>Loading..</p>

  return null
}
```

このコードは、データが取得されるまでローディングスピナーを表示し、データが取得されたらそのデータを初期値としてフォームをレンダリングします。
