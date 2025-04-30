---
source-updated-at: '2025-03-14T21:54:05.000Z'
translation-updated-at: '2025-04-30T21:27:32.961Z'
id: basic-concepts
title: 基本概念
---

このページでは、`@tanstack/solid-form` ライブラリで使用される基本的な概念と用語を紹介します。これらの概念を理解することで、ライブラリをより効果的に活用し、複雑なフォームを簡単に作成できるようになります。

## フォームオプション

`formOptions` 関数を使用して、複数のフォーム間で共有可能なフォームオプションを作成できます。

例:

```tsx
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## フォームインスタンス

フォームインスタンスは、個々のフォームを表すオブジェクトで、フォームを操作するためのメソッドとプロパティを提供します。フォームインスタンスは、フォームオプションが提供する `createForm` フックを使用して作成します。このフックは、フォームが送信されたときに呼び出される `onSubmit` 関数を含むオブジェクトを受け取ります。

```tsx
const form = createForm(() => ({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // フォームデータを処理
    console.log(value)
  },
}))
```

`formOptions` を使用せずに、スタンドアロンの `createForm` API を使用してフォームインスタンスを作成することもできます:

```tsx
const form = createForm<Person>(() => ({
  onSubmit: async ({ value }) => {
    // フォームデータを処理
    console.log(value)
  },
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  },
}))
```

## フィールド

フィールドは、テキスト入力やチェックボックスなどの単一のフォーム入力要素を表します。フィールドは、フォームインスタンスが提供する `form.Field` コンポーネントを使用して作成します。このコンポーネントは、フォームのデフォルト値のキーと一致する `name` プロパティと、フィールドオブジェクトを引数とするレンダープロップ関数である `children` プロパティを受け取ります。

例:

```tsx
<form.Field
  name="firstName"
  children={(field) => (
    <input
      name={field().name}
      value={field().state.value}
      onBlur={field().handleBlur}
      onInput={(e) => field().handleChange(e.target.value)}
    />
  )}
/>
```

## フィールドステート

各フィールドには、現在の値、検証ステータス、エラーメッセージ、およびその他のメタデータを含む独自のステートがあります。フィールドのステートには `field().state` プロパティを使用してアクセスできます。

例:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field().state
```

ユーザーがフィールドとどのようにやり取りしているかを確認するのに役立つ3つのフィールドステートがあります。フィールドは、ユーザーがクリック/タブでフォーカスを当てると「touched (触れた)」、値が変更されるまでは「pristine (未変更)」、値が変更されると「dirty (変更済み)」とマークされます。これらのステートは、以下のように `isTouched`、`isPristine`、`isDirty` フラグで確認できます。

```tsx
const { isTouched, isPristine, isDirty } = field().state.meta
```

![フィールドステート](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## フィールドAPI

フィールドAPIは、フィールドを作成するときにレンダープロップ関数に渡されるオブジェクトで、フィールドのステートを操作するためのメソッドを提供します。

例:

```tsx
<input
  name={field().name}
  value={field().state.value}
  onBlur={field().handleBlur}
  onInput={(e) => field().handleChange(e.target.value)}
/>
```

## バリデーション

`@tanstack/solid-form` は、同期および非同期バリデーションを標準で提供しています。バリデーション関数は、`validators` プロパティを使用して `form.Field` コンポーネントに渡すことができます。

例:

```tsx
<form.Field
  name="firstName"
  validators={{
    onChange: ({ value }) =>
      !value
        ? 'A first name is required'
        : value.length < 3
          ? 'First name must be at least 3 characters'
          : undefined,
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && 'No "error" allowed in first name'
    },
  }}
  children={(field) => (
    <>
      <input
        name={field().name}
        value={field().state.value}
        onBlur={field().handleBlur}
        onInput={(e) => field().handleChange(e.target.value)}
      />
      <p>{field().state.meta.errors[0]}</p>
    </>
  )}
/>
```

## 標準スキーマライブラリを使用したバリデーション

手動で作成したバリデーションオプションに加えて、[Standard Schema](https://github.com/standard-schema/standard-schema) 仕様を実装したライブラリもサポートしています。

仕様を実装した任意のライブラリを使用してスキーマを定義し、フォームまたはフィールドバリデータに渡すことができます。

サポートされているライブラリ:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```tsx
import { z } from 'zod'

// ...
;<form.Field
  name="firstName"
  validators={{
    onChange: z.string().min(3, 'First name must be at least 3 characters'),
    onChangeAsyncDebounceMs: 500,
    onChangeAsync: z.string().refine(
      async (value) => {
        await new Promise((resolve) => setTimeout(resolve, 1000))
        return !value.includes('error')
      },
      {
        message: 'No "error" allowed in first name',
      },
    ),
  }}
  children={(field) => (
    <>
      <input
        name={field().name}
        value={field().state.value}
        onBlur={field().handleBlur}
        onInput={(e) => field().handleChange(e.target.value)}
      />
      <p>{field().state.meta.errors[0]}</p>
    </>
  )}
/>
```

## リアクティビティ

`@tanstack/solid-form` は、フォームとフィールドのステート変更をサブスクライブするためのさまざまな方法を提供しており、特に `form.useStore` フックと `form.Subscribe` コンポーネントが注目されます。これらのメソッドを使用すると、必要な時だけコンポーネントを更新することで、フォームのレンダリングパフォーマンスを最適化できます。

例:

```tsx
const firstName = form.useStore((state) => state.values.firstName)
//...
<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
  children={(state) => (
    <button type="submit" disabled={!state().canSubmit}>
      {state().isSubmitting ? '...' : 'Submit'}
    </button>
  )}
/>
```

## 配列フィールド

配列フィールドを使用すると、趣味のリストなどのフォーム内の値のリストを管理できます。配列フィールドは、`mode="array"` プロパティを指定した `form.Field` コンポーネントを使用して作成できます。

配列フィールドを操作する際には、`pushValue`、`removeValue`、`swapValues`、`moveValue` メソッドを使用して、配列内の値を追加、削除、入れ替えできます。

例:

```tsx
<form.Field
  name="hobbies"
  mode="array"
  children={(hobbiesField) => (
    <div>
      Hobbies
      <div>
        <Show
          when={hobbiesField().state.value.length > 0}
          fallback={'No hobbies found.'}
        >
          <Index each={hobbiesField().state.value}>
            {(_, i) => (
              <div>
                <form.Field
                  name={`hobbies[${i}].name`}
                  children={(field) => (
                    <div>
                      <label for={field().name}>Name:</label>
                      <input
                        id={field().name}
                        name={field().name}
                        value={field().state.value}
                        onBlur={field().handleBlur}
                        onInput={(e) => field().handleChange(e.target.value)}
                      />
                      <button
                        type="button"
                        onClick={() => hobbiesField().removeValue(i)}
                      >
                        X
                      </button>
                    </div>
                  )}
                />
                <form.Field
                  name={`hobbies[${i}].description`}
                  children={(field) => {
                    return (
                      <div>
                        <label for={field().name}>Description:</label>
                        <input
                          id={field().name}
                          name={field().name}
                          value={field().state.value}
                          onBlur={field().handleBlur}
                          onInput={(e) => field().handleChange(e.target.value)}
                        />
                      </div>
                    )
                  }}
                />
              </div>
            )}
          </Index>
        </Show>
      </div>
      <button
        type="button"
        onClick={() =>
          hobbiesField().pushValue({
            name: '',
            description: '',
            yearsOfExperience: 0,
          })
        }
      >
        Add hobby
      </button>
    </div>
  )}
/>
```

以上が、`@tanstack/solid-form` ライブラリで使用される基本的な概念と用語です。これらの概念を理解することで、ライブラリをより効果的に活用し、複雑なフォームを簡単に作成できるようになります。
