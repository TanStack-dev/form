---
source-updated-at: '2025-03-20T03:01:05.000Z'
translation-updated-at: '2025-04-30T21:25:03.854Z'
id: reactivity
title: リアクティビティ
---

# リアクティビティ (Reactivity)

Tanstack Formはフォームとのインタラクション時に再レンダリングを引き起こしません。そのため、フォームやフィールドの状態値をうまく利用できない場合があります。

リアクティブな値にアクセスしたい場合は、以下の2つの方法のいずれかを使ってサブスクライブする必要があります: `useStore` または `form.Subscribe` コンポーネントです。

これらのサブスクリプションの使用例としては、最新のフィールド値をレンダリングする、条件に基づいてレンダリングする内容を決定する、コンポーネントのロジック内でフィールド値を使用するなどがあります。

> トリガーに「反応」したい場合は、[リスナー (listener)](./listeners.md) APIを確認してください。

## useStore

`useStore` フックは、コンポーネントのロジック内でフォーム値にアクセスする必要がある場合に最適です。`useStore` は2つのパラメータを取ります。1つ目はフォームストア、2つ目はサブスクライブしたいフォームの部分を細かく調整するセレクターです。

```tsx
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
```

セレクター内でフォーム状態の任意の部分にアクセスできます。

> 注意: `useStore` は、サブスクライブしている値が変更されるたびにコンポーネント全体の再レンダリングを引き起こします。

セレクターを省略することは可能ですが、フォーム状態が変更されるたびに不要な再レンダリングが多数発生するため、省略しないようにしてください。

## form.Subscribe

`form.Subscribe` コンポーネントは、コンポーネントのUI内で何かに反応する必要がある場合に最適です。例えば、フォームフィールドの値に基づいてUIを表示または非表示にする場合などです。

```tsx
<form.Subscribe
  selector={(state) => state.values.firstName}
  children={(firstName) => (
    <form.Field>
      {(field) => (
        <input
          name="lastName"
          value={field.state.lastName}
          onChange={field.handleChange}
        />
      )}
    </form.Field>
  )}
/>
```

> `form.Subscribe` コンポーネントはコンポーネントレベルの再レンダリングを引き起こしません。サブスクライブしている値が変更されるたびに、`form.Subscribe` コンポーネントのみが再レンダリングされます。

`useStore` と `form.Subscribe` のどちらを使用するかは、主に以下のように判断します: UIにレンダリングされる場合は最適化の利点がある `form.Subscribe` を選択し、ロジック内でリアクティビティが必要な場合は `useStore` を選択します。
