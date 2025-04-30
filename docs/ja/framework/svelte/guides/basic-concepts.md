---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:28:43.995Z'
id: basic-concepts
title: 基本概念
---

このページでは、`@tanstack/svelte-form`ライブラリで使用される基本的な概念と用語を紹介します。これらの概念を理解することで、ライブラリをより効果的に扱えるようになります。

## フォームオプション

`formOptions`関数を使用して、複数のフォーム間で共有可能なオプションを作成できます。

例:

```ts
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## フォームインスタンス

フォームインスタンスは個々のフォームを表すオブジェクトで、フォームを操作するためのメソッドやプロパティを提供します。`createForm`関数を使用してフォームインスタンスを作成します。この関数は`onSubmit`関数を含むオブジェクトを受け取り、フォームが送信されると呼び出されます。

```ts
const form = createForm(() => ({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // フォームデータで何か処理を行う
    console.log(value)
  },
}))
```

`formOptions`を使用せずにフォームインスタンスを作成することも可能です:

```ts
const form = createForm<Person>(() => ({
  onSubmit: async ({ value }) => {
    // フォームデータで何か処理を行う
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

フィールドは、テキスト入力やチェックボックスなどの単一のフォーム入力要素を表します。フィールドはフォームインスタンスが提供する`form.Field`コンポーネントを使用して作成されます。このコンポーネントは、フォームのデフォルト値のキーと一致する必要がある`name`プロパティと、フィールドオブジェクトを引数に取るレンダープロップ関数である`children`プロパティを受け取ります。

例:

```svelte
<form.Field name="firstName">
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onblur={field.handleBlur}
      oninput={(e) => field.handleChange(e.target.value)}
    />
  {/snippet}
</form.Field>
```

## フィールド状態

各フィールドには、現在の値、検証ステータス、エラーメッセージ、その他のメタデータを含む独自の状態があります。`field.state`プロパティを使用してフィールドの状態にアクセスできます。

例:

```ts
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

ユーザーがフィールドとどのようにやり取りしているかを確認するのに役立つ3つのフィールド状態があります。フィールドは、ユーザーがクリック/タブでフォーカスすると「touched」、値が変更されるまでは「pristine」、値が変更された後は「dirty」とマークされます。以下のように`isTouched`、`isPristine`、`isDirty`フラグでこれらの状態を確認できます。

```ts
const { isTouched, isPristine, isDirty } = field.state.meta
```

![フィールド状態](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## フィールドAPI

フィールドAPIは、フィールドを作成する際にレンダープロップ関数に渡されるオブジェクトで、フィールドの状態を操作するためのメソッドを提供します。

例:

```svelte
<input
  name={field.name}
  value={field.state.value}
  onblur={field.handleBlur}
  oninput={(e) => field.handleChange(e.target.value)}
/>
```

## バリデーション

`@tanstack/svelte-form`は、同期および非同期のバリデーションを標準で提供しています。バリデーション関数は`form.Field`コンポーネントの`validators`プロパティに渡すことができます。

例:

```svelte
<form.Field
  name="firstName"
  validators={{
    onChange: ({ value }) =>
      !value
        ? '名は必須です'
        : value.length < 3
          ? '名は3文字以上必要です'
          : undefined,
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && '名に"error"を含めることはできません'
    },
  }}
>
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onBlur={field.handleBlur}
      onInput={(e) => field.handleChange(e.target.value)}
    />
    <p>{field.state.meta.errors[0]}</p>
  {/snippet}
</form.Field>
```

## 標準スキーマライブラリによるバリデーション

独自のバリデーションオプションに加えて、[Standard Schema](https://github.com/standard-schema/standard-schema)仕様もサポートしています。

この仕様を実装した任意のライブラリを使用してスキーマを定義し、フォームまたはフィールドバリデータに渡すことができます。

サポートされているライブラリ:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```svelte
<script>
  import { z } from 'zod'

  // ...
</script>

<form.Field
  name="firstName"
  validators={{
    onChange: z.string().min(3, '名は3文字以上必要です'),
    onChangeAsyncDebounceMs: 500,
    onChangeAsync: z.string().refine(
      async (value) => {
        await new Promise((resolve) => setTimeout(resolve, 1000))
        return !value.includes('error')
      },
      {
        message: '名に"error"を含めることはできません',
      },
    ),
  }}
>
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onBlur={field.handleBlur}
      onInput={(e) => field.handleChange(e.target.value)}
    />
    <p>{field.state.meta.errors[0]}</p>
  {/snippet}
</form.Field>
```

## リアクティビティ

`@tanstack/svelte-form`は、フォームとフィールドの状態変更を購読するさまざまな方法を提供しており、特に`form.useStore`フックと`form.Subscribe`コンポーネントが注目されます。これらのメソッドを使用すると、必要な時だけコンポーネントを更新することでフォームのレンダリングパフォーマンスを最適化できます。

例:

```svelte
<script>
  //...
  const firstName = form.useStore((state) => state.values.firstName)
</script>

<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
>
  {#snippet children(state)}
    <button type="submit" disabled={!state.canSubmit}>
      {state.isSubmitting ? '...' : '送信'}
    </button>
  {/snippet}
</form.Subscribe>
```

## 配列フィールド

配列フィールドを使用すると、趣味のリストなど、フォーム内の値のリストを管理できます。`mode="array"`プロパティを指定した`form.Field`コンポーネントを使用して配列フィールドを作成できます。

配列フィールドを操作する際は、`pushValue`、`removeValue`、`swapValues`、`moveValue`メソッドを使用して配列内の値を追加、削除、交換できます。

例:

```svelte
<form.Field name="hobbies" mode="array">
  {#snippet children(hobbiesField)}
    <div>
      趣味
      <div>
        {#each hobbiesField.state.value as _, i}
            <div>
              <form.Field name={`hobbies[${i}].name`}>
                {#snippet children(field)}
                  <div>
                    <label for={field.name}>名前:</label>
                    <input
                      id={field.name}
                      name={field.name}
                      value={field.state.value}
                      onblur={field.handleBlur}
                      onchange={(e) => field.handleChange(e.target.value)}
                    />
                    <button
                      type="button"
                      onclick={() => hobbiesField.removeValue(i)}
                    >
                      X
                    </button>
                  </div>
                {/snippet}
              </form.Field>
              <form.Field name={`hobbies[${i}].description`}>
                {#snippet children(field)}
                    <div>
                      <label for={field.name}>説明:</label>
                      <input
                        id={field.name}
                        name={field.name}
                        value={field.state.value}
                        onblur={field.handleBlur}
                        onchange={(e) => field.handleChange(e.target.value)}
                      />
                    </div>
                {/snippet}
              </form.Field>
            </div>
          {:else}
            趣味が登録されていません。
          {/each}
      </div>
      <button
        type="button"
        onclick={() =>
          hobbiesField.pushValue({
            name: '',
            description: '',
            yearsOfExperience: 0,
          })
        }
      >
        趣味を追加
      </button>
    </div>
  {/snippet}
</form.Field>
```

これらが`@tanstack/svelte-form`ライブラリで使用される基本的な概念と用語です。これらの概念を理解することで、ライブラリをより効果的に使用し、複雑なフォームを簡単に作成できるようになります。
