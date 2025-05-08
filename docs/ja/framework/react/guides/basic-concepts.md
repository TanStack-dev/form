---
source-updated-at: '2025-05-08T07:42:29.000Z'
translation-updated-at: '2025-05-08T23:45:38.105Z'
id: basic-concepts
title: 基本概念
---

このページでは、`@tanstack/react-form` ライブラリで使用される基本概念と用語を紹介します。これらの概念を理解することで、ライブラリをより効果的に活用できるようになります。

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

フォームインスタンスは個々のフォームを表現するオブジェクトで、フォームを操作するためのメソッドやプロパティを提供します。フォームインスタンスは、フォームオプションから提供される `useForm` フックを使用して作成します。このフックは、フォームが送信されたときに呼び出される `onSubmit` 関数を含むオブジェクトを受け取ります。

```tsx
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // フォームデータで何か処理を行う
    console.log(value)
  },
})
```

`formOptions` を使用せずにスタンドアロンの `useForm` API を使用してフォームインスタンスを作成することもできます:

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
    // フォームデータで何か処理を行う
    console.log(value)
  },
})
```

## フィールド

フィールドは、テキスト入力やチェックボックスなどの単一のフォーム入力要素を表します。フィールドは、フォームインスタンスから提供される `form.Field` コンポーネントを使用して作成します。このコンポーネントは、フォームのデフォルト値のキーと一致する `name` プロパティと、フィールドオブジェクトを引数として受け取るレンダープロップ関数である `children` プロパティを受け取ります。

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

各フィールドには、現在の値、検証ステータス、エラーメッセージ、その他のメタデータを含む独自の状態があります。フィールドの状態には `field.state` プロパティを使用してアクセスできます。

例:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

ユーザーがフィールドとどのようにやり取りしているかを確認するのに役立つ、メタデータ内の3つの状態があります:

- _"isTouched"_: ユーザーがフィールドをクリック/タップした後
- _"isPristine"_: ユーザーがフィールドの値を変更するまで
- _"isDirty"_: フィールドの値が変更された後

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![フィールド状態](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## 異なるライブラリにおける 'isDirty' の理解

非永続的な `dirty` 状態

- **ライブラリ**: React Hook Form (RHF), Formik, Final Form
- **動作**: フィールドの値がデフォルトと異なる場合に 'dirty' となる。デフォルト値に戻すと 'clean' に戻る

永続的な `dirty` 状態

- **ライブラリ**: Angular Form, Vue FormKit
- **動作**: 一度変更されると、デフォルト値に戻しても 'dirty' のまま

私たちは永続的な 'dirty' 状態モデルを選択しました。非永続的な 'dirty' 状態もサポートするために、`isDefault` フラグを導入しています。このフラグは非永続的な 'dirty' 状態の逆として機能します。

```tsx
const { isTouched, isPristine, isDirty, isDefaultValue } = field.state.meta

// 以下の行で非永続的な `dirty` 機能を再現できます
const nonPersistentIsDirty = !isDefaultValue
```

![拡張されたフィールド状態](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states-extended.png)

## フィールドAPI

フィールドAPIは、フィールドを作成するときにレンダープロップ関数に渡されるオブジェクトで、フィールドの状態を操作するためのメソッドを提供します。

例:

```tsx
<input
  value={field.state.value}
  onBlur={field.handleBlur}
  onChange={(e) => field.handleChange(e.target.value)}
/>
```

## バリデーション

`@tanstack/react-form` は、同期および非同期バリデーションを標準で提供しています。バリデーション関数は、`validators` プロパティを使用して `form.Field` コンポーネントに渡すことができます。

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
      return value.includes('error') && '名に"error"を含めることはできません'
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

## 標準スキーマライブラリによるバリデーション

手動で作成したバリデーションオプションに加えて、[Standard Schema](https://github.com/standard-schema/standard-schema) 仕様もサポートしています。

仕様を実装した任意のライブラリを使用してスキーマを定義し、フォームまたはフィールドバリデータに渡すことができます。

サポートされているライブラリ:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```tsx
import { z } from 'zod'

const userSchema = z.object({
  age: z.number().gte(13, 'アカウント作成には13歳以上である必要があります'),
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

`@tanstack/react-form` は、フォームやフィールドの状態変化をサブスクライブするさまざまな方法を提供しており、特に `useStore(form.store)` フックと `form.Subscribe` コンポーネントが注目されます。これらの方法を使用すると、必要な時だけコンポーネントを更新することで、フォームのレンダリングパフォーマンスを最適化できます。

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

`useStore` フックの `selector` プロパティはオプションですが、省略すると不要な再レンダリングが発生するため、提供することを強く推奨します。

```tsx
// 正しい使い方
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
// 間違った使い方
const store = useStore(form.store)
```

注: リアクティビティを実現するための `useField` フックの使用は推奨されません。このフックは `form.Field` コンポーネント内で慎重に使用するように設計されています。代わりに `useStore(form.store)` を使用することを検討してください。

## リスナー

`@tanstack/react-form` では、特定のトリガーに反応して副作用をディスパッチする「リスナー」を設定できます。

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

配列フィールドを使用すると、趣味のリストなどの値のリストをフォーム内で管理できます。配列フィールドは、`mode="array"` プロパティを指定した `form.Field` コンポーネントを使用して作成します。

配列フィールドを操作する際には、`pushValue`、`removeValue`、`swapValues`、`moveValue` メソッドを使用して、配列内の値を追加、削除、交換できます。

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

`<button type="reset">` を TanStack Form の `form.reset()` と組み合わせて使用する場合、特に `<select>` 要素が初期HTML値に予期せずリセットされるのを防ぐために、デフォルトのHTMLリセット動作を防ぐ必要があります。
ボタンの `onClick` ハンドラ内で `event.preventDefault()` を使用して、ネイティブのフォームリセットを防ぎます。

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

これらは `@tanstack/react-form` ライブラリで使用される基本概念と用語です。これらの概念を理解することで、ライブラリをより効果的に使用し、複雑なフォームを簡単に作成できるようになります。
