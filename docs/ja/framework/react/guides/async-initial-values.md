---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:24:46.181Z'
id: async-initial-values
title: 非同期初期値
---

APIからデータを取得し、それをフォームの初期値として使用したい場合を考えてみましょう。

この問題は表面上は単純に見えますが、これまで気づかなかった複雑な要素が潜んでいる可能性があります。

例えば、データ取得中にローディングスピナーを表示したい場合や、エラーを適切に処理したい場合があります。
同様に、フォームがレンダリングされるたびにデータを取得しなくて済むように、データをキャッシュする方法を探しているかもしれません。

これらの機能の多くを一から実装することも可能ですが、結果的には私たちがメンテナンスしている別のプロジェクトである [TanStack Query](https://tanstack.com/query) と非常に似たものになるでしょう。

したがって、このガイドでは、TanStack Form と TanStack Query を組み合わせて目的の動作を実現する方法を説明します。

## 基本的な使い方

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

このコードは、データが取得されるまでローディングスピナーを表示し、その後取得したデータを初期値としてフォームをレンダリングします。
