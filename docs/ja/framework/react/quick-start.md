---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T21:24:50.145Z'
id: quick-start
title: クイックスタート
---

TanStack Formは、これまで使ってきたフォームライブラリとは一線を画しています。大規模なプロダクション環境での使用を想定して設計されており、タイプセーフティ、パフォーマンス、コンポジションに重点を置くことで、他に類を見ない開発者体験を実現しています。

その結果、私たちは[ライブラリの使用に関する哲学](/form/latest/docs/philosophy)を確立しました。これは、短く共有可能なコードスニペットよりも、スケーラビリティと長期的な開発者体験を重視するものです。

以下は、私たちのベストプラクティスの多くを取り入れたフォームの例です。これにより、短期間のオンボーディング後でも、高度に複雑なフォームを迅速に開発できるようになります:

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { createFormHook, createFormHookContexts } from '@tanstack/react-form'
// フォームフックからイベントを事前にバインドするフォームコンポーネント。詳細は「Form Composition」ガイドを参照
import { TextField, NumberField, SubmitButton } from '~our-app/ui-library'
// Valibot、ArkType、その他の標準スキーマライブラリもサポート
import { z } from 'zod'

const { fieldContext, formContext } = createFormHookContexts()

// タイプセーフティを保ちつつ、プロダクション環境でのボイラープレートを削減するためにコンポーネントをフォームにバインド
// これを一度定義すれば、アプリ全体で一貫したフォームインスタンスを生成可能
const { useAppForm } = createFormHook({
  fieldComponents: {
    TextField,
    NumberField,
  },
  formComponents: {
    SubmitButton,
  },
  fieldContext,
  formContext,
})

const PeoplePage = () => {
  const form = useAppForm({
    defaultValues: {
      username: '',
      age: 0,
    },
    validators: {
      // スキーマまたは検証関数を渡す
      onChange: z.object({
        username: z.string(),
        age: z.number().min(13),
      }),
    },
    onSubmit: ({ value }) => {
      // フォームデータで何か処理を行う
      alert(JSON.stringify(value, null, 2))
    },
  })

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault()
        form.handleSubmit()
      }}
    >
      <h1>Personal Information</h1>
      {/* コンポーネントは`form`と`field`にバインドされ、極めて高いタイプセーフティを確保 */}
      {/* `form.AppField`を使用して、単一フィールドにバインドされたコンポーネントをレンダリング */}
      <form.AppField
        name="username"
        children={(field) => <field.TextField label="Full Name" />}
      />
      {/* "name"プロパティにタイポがあるとTypeScriptエラーが発生 */}
      <form.AppField
        name="age"
        children={(field) => <field.NumberField label="Age" />}
      />
      {/* `form.AppForm`内のコンポーネントはフォームコンテキストにアクセス可能 */}
      <form.AppForm>
        <form.SubmitButton />
      </form.AppForm>
    </form>
  )
}

const rootElement = document.getElementById('root')!
ReactDOM.createRoot(rootElement).render(<PeoplePage />)
```

長期的にはボイラープレートを削減するため`createFormHook`の使用を推奨しますが、`useForm`と`form.Field`を使用したワンオフコンポーネントやその他の動作もサポートしています:

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { useForm } from '@tanstack/react-form'

const PeoplePage = () => {
  const form = useForm({
    defaultValues: {
      username: '',
      age: 0,
    },
    onSubmit: ({ value }) => {
      // フォームデータで何か処理を行う
      alert(JSON.stringify(value, null, 2))
    },
  })

  return (
    <form.Field
      name="age"
      validators={{
        // フォーム全体またはフィールド固有のバリデータを選択可能
        onChange: ({ value }) =>
          value > 13 ? undefined : 'Must be 13 or older',
      }}
      children={(field) => (
        <>
          <input
            name={field.name}
            value={field.state.value}
            onBlur={field.handleBlur}
            type="number"
            onChange={(e) => field.handleChange(e.target.valueAsNumber)}
          />
          {!field.state.meta.isValid && (
            <em>{field.state.meta.errors.join(',')}</em>
          )}
        </>
      )}
    />
  )
}

const rootElement = document.getElementById('root')!
ReactDOM.createRoot(rootElement).render(<PeoplePage />)
```

`useForm`のすべてのプロパティは`useAppForm`で使用でき、`form.Field`のすべてのプロパティは`form.AppField`で使用できます。
