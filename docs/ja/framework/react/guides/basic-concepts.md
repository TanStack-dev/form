---
source-updated-at: '2025-04-16T08:45:06.000Z'
translation-updated-at: '2025-04-30T21:26:06.581Z'
id: basic-concepts
title: 基本概念
---

このページでは、`@tanstack/react-form` ライブラリで使用される基本的な概念と用語を紹介します。これらの概念を理解することで、ライブラリをより効果的に活用できるようになります。

## フォームオプション

`formOptions` 関数を使用して、複数のフォーム間で共有可能なオプションを作成できます。

例:

```tsx
interface User {
  firstName: string
  lastName: string
  hobbies: Array<string>
}
const defaultUser: User = { firstName: '', lastName: '', hobbies: [] }

const formOpts = formOptions({
  defaultValues: defaultUser,
})
```

## フォームインスタンス

フォームインスタンスは個々のフォームを表すオブジェクトで、フォームを操作するためのメソッドとプロパティを提供します。フォームインスタンスは、フォームオプションが提供する `useForm` フックを使用して作成します。このフックは、フォームが送信されたときに呼び出される `onSubmit` 関数を含むオブジェクトを受け取ります。

```tsx
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // フォームデータを処理
    console.log(value)
  },
})
```

`formOptions` を使用せずにスタンドアロンの `useForm` API でフォームインスタンスを作成することも可能です:

```tsx
interface User {
  firstName: string
  lastName: string
  hobbies: Array<string>
}
const defaultUser: User = { firstName: '', lastName: '', hobbies: [] }

const form = useForm({
  defaultValues: defaultUser,
  onSubmit: async ({ value }) => {
    // フォームデータを処理
    console.log(value)
  },
})
```

## フィールド

フィールドは、テキスト入力やチェックボックスなどの単一のフォーム入力要素を表します。フィールドは、フォームインスタンスが提供する `form.Field` コンポーネントを使用して作成します。このコンポーネントは、フォームのデフォルト値のキーと一致する `name` プロパティと、フィールドオブジェクトを引数に取るレンダープロップ関数である `children` プロパティを受け取ります。

例:

```tsx
<form.Field
  name="firstName"
  children={(field) => (
    <>
      <input
        value={field.state.value}
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.value)}
      />
      <FieldInfo field={field} />
    </>
  )}
/>
```

## フィールド状態

各フィールドには、現在の値、検証ステータス、エラーメッセージなどのメタデータを含む独自の状態があります。フィールドの状態には `field.state` プロパティでアクセスできます。

例:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

ユーザーがフィールドとどのようにやり取りしているかを確認するのに役立つ3つのフィールド状態があります: ユーザーがクリック/タブでフィールドに入ると「touched」、値が変更されるまでは「pristine」、値が変更されると「dirty」になります。これらの状態は、以下のように `isTouched`、`isPristine`、`isDirty` フラグで確認できます。

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![フィールド状態](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

> **`React Hook Form` からの移行に関する重要な注意**: `TanStack/form` の `isDirty` フラグは、RHF の同名フラグとは異なります。
> RHF では、フォームの値が元の値と異なる場合に `isDirty = true` になります。ユーザーがフォームの値を変更し、再度変更してフォームのデフォルト値と一致する値になった場合、RHF では `isDirty` は `false` になりますが、`TanStack/form` では `true` のままです。
> デフォルト値は `TanStack/form` のフォームレベルとフィールドレベルで公開されています（`form.options.defaultValues`、`field.options.defaultValue`）。RHF の動作をエミュレートする必要がある場合は、独自の `isDefaultValue()` ヘルパーを記述できます。

## フィールドAPI

フィールドAPIは、フィールドを作成する際にレンダープロップ関数に渡されるオブジェクトで、フィールドの状態を操作するためのメソッドを提供します。

例:

```tsx
<input
  value={field.state.value}
  onBlur={field.handleBlur}
  onChange={(e) => field.handleChange(e.target.value)}
/>
```

## バリデーション

`@tanstack/react-form` は、同期および非同期のバリデーションを標準で提供しています。バリデーション関数は `form.Field` コンポーネントの `validators` プロパティに渡すことができます。

例:

```tsx
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
      return value.includes('error') && '名に「error」を含めることはできません'
    },
  }}
  children={(field) => (
    <>
      <input
        value={field.state.value}
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.value)}
      />
      <FieldInfo field={field} />
    </>
  )}
/>
```

## 標準スキーマライブラリを使用したバリデーション

手動で作成したバリデーションオプションに加えて、[Standard Schema](https://github.com/standard-schema/standard-schema) 仕様もサポートしています。

この仕様を実装した任意のライブラリを使用してスキーマを定義し、フォームまたはフィールドのバリデータに渡すことができます。

サポートされているライブラリ:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```tsx
import { z } from 'zod'

const userSchema = z.object({
  age: z
    .number()
    .gte(13, 'アカウントを作成するには13歳以上である必要があります'),
})

function App() {
  const form = useForm({
    defaultValues: {
      age: 0,
    },
    validators: {
      onChange: userSchema,
    },
  })
  return (
    <div>
      <form.Field
        name="age"
        children={(field) => {
          return <>{/* ... */}</>
        }}
      />
    </div>
  )
}
```

## リアクティビティ

`@tanstack/react-form` は、フォームとフィールドの状態変更をサブスクライブするためのさまざまな方法を提供しており、特に `useStore(form.store)` フックと `form.Subscribe` コンポーネントが注目されます。これらの方法を使用すると、必要な時だけコンポーネントを更新することで、フォームのレンダリングパフォーマンスを最適化できます。

例:

```tsx
const firstName = useStore(form.store, (state) => state.values.firstName)
//...
<form.Subscribe
  selector={(state) => [state.canSubmit, state.isSubmitting]}
  children={([canSubmit, isSubmitting]) => (
    <button type="submit" disabled={!canSubmit}>
      {isSubmitting ? '...' : '送信'}
    </button>
  )}
/>
```

`useStore` フックの `selector` プロパティはオプションですが、これを省略すると不要な再レンダリングが発生するため、強く推奨される点に注意してください。

```tsx
// 正しい使用法
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
// 誤った使用法
const store = useStore(form.store)
```

注: リアクティビティを実現するための `useField` フックの使用は推奨されません。このフックは `form.Field` コンポーネント内で慎重に使用するように設計されています。代わりに `useStore(form.store)` の使用を検討してください。

## リスナー

`@tanstack/react-form` では、特定のトリガーに反応してサイドエフェクトをディスパッチする「リスナー」を設定できます。

例:

```tsx
<form.Field
  name="country"
  listeners={{
    onChange: ({ value }) => {
      console.log(`国が ${value} に変更されました、州をリセットします`)
      form.setFieldValue('province', '')
    },
  }}
/>
```

詳細は [リスナー](./listeners.md) を参照してください。

## 配列フィールド

配列フィールドを使用すると、趣味のリストなど、フォーム内の値のリストを管理できます。配列フィールドは `mode="array"` プロパティを指定した `form.Field` コンポーネントを使用して作成します。

配列フィールドを操作する際は、`pushValue`、`removeValue`、`swapValues`、`moveValue` メソッドを使用して、配列内の値を追加、削除、入れ替えできます。

例:

```tsx
<form.Field
  name="hobbies"
  mode="array"
  children={(hobbiesField) => (
    <div>
      趣味
      <div>
        {!hobbiesField.state.value.length
          ? '趣味がありません'
          : hobbiesField.state.value.map((_, i) => (
              <div key={i}>
                <form.Field
                  name={`hobbies[${i}].name`}
                  children={(field) => {
                    return (
                      <div>
                        <label htmlFor={field.name}>名前:</label>
                        <input
                          id={field.name}
                          name={field.name}
                          value={field.state.value}
                          onBlur={field.handleBlur}
                          onChange={(e) => field.handleChange(e.target.value)}
                        />
                        <button
                          type="button"
                          onClick={() => hobbiesField.removeValue(i)}
                        >
                          X
                        </button>
                        <FieldInfo field={field} />
                      </div>
                    )
                  }}
                />
                <form.Field
                  name={`hobbies[${i}].description`}
                  children={(field) => {
                    return (
                      <div>
                        <label htmlFor={field.name}>説明:</label>
                        <input
                          id={field.name}
                          name={field.name}
                          value={field.state.value}
                          onBlur={field.handleBlur}
                          onChange={(e) => field.handleChange(e.target.value)}
                        />
                        <FieldInfo field={field} />
                      </div>
                    )
                  }}
                />
              </div>
            ))}
      </div>
      <button
        type="button"
        onClick={() =>
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
  )}
/>
```

## リセットボタン

`<button type="reset">` を TanStack Form の `form.reset()` と併用する場合、HTMLのデフォルトのリセット動作を防ぐ必要があります（特に `<select>` 要素が初期HTML値にリセットされるのを避けるため）。ボタンの `onClick` ハンドラー内で `event.preventDefault()` を使用して、ネイティブのフォームリセットを防ぎます。

例:

```tsx
<button
  type="reset"
  onClick={(event) => {
    event.preventDefault()
    form.reset()
  }}
>
  リセット
</button>
```

または、ネイティブのHTMLリセットを防ぐために `<button type="button">` を使用することもできます。

```tsx
<button
  type="button"
  onClick={() => {
    form.reset()
  }}
>
  リセット
</button>
```

これらは `@tanstack/react-form` ライブラリで使用される基本的な概念と用語です。これらの概念を理解することで、ライブラリをより効果的に活用し、複雑なフォームを簡単に作成できるようになります。
