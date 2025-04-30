---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:26:49.544Z'
id: form-composition
title: フォーム構成
---

# フォーム構成

TanStack Formに対する一般的な批判は、デフォルトでの冗長さです。これは教育目的には有用（APIの理解を強化するため）かもしれませんが、実際のプロダクション環境での使用には理想的ではありません。

その結果、`form.Field`がTanStack Formの最も強力で柔軟な使用方法を可能にする一方で、それをラップしてアプリケーションコードをより簡潔にするAPIも提供しています。

## カスタムフォームフック

フォームを構成する最も強力な方法は、カスタムフォームフックを作成することです。これにより、アプリケーションのニーズに合わせたフォームフックを作成でき、事前にバインドされたカスタムUIコンポーネントなどを含めることができます。

最も基本的な形では、`createFormHook`は`fieldContext`と`formContext`を受け取り、`useAppForm`フックを返す関数です。

> このカスタマイズされていない`useAppForm`フックは`useForm`と同じですが、`createFormHook`にオプションを追加することですぐに変化します。

```tsx
import { createFormHookContexts, createFormHook } from '@tanstack/react-form'

// カスタムコンポーネントで使用するためにuseFieldContextをエクスポート
export const { fieldContext, formContext, useFieldContext } =
  createFormHookContexts()

const { useAppForm } = createFormHook({
  fieldContext,
  formContext,
  // これらのオプションについては後で詳しく説明します
  fieldComponents: {},
  formComponents: {},
})

function App() {
  const form = useAppForm({
    // useFormのすべてのオプションをサポート
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return <form.Field /> // ...
}
```

### 事前バインドされたフィールドコンポーネント

このスキャフォールディングが整ったら、フォームフックにカスタムフィールドやフォームコンポーネントを追加できます。

> 注意: `useFieldContext`はカスタムフォームコンテキストからエクスポートされたものと同じでなければなりません

```tsx
import { useFieldContext } from './form-context.tsx'

export function TextField({ label }: { label: string }) {
  // `Field`は`string`型の`value`を持つべきだと推論します
  const field = useFieldContext<string>()
  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}
```

次に、このコンポーネントをフォームフックに登録できます。

```tsx
import { TextField } from './text-field.tsx'

const { useAppForm } = createFormHook({
  fieldContext,
  formContext,
  fieldComponents: {
    TextField,
  },
  formComponents: {},
})
```

そしてフォーム内で使用します:

```tsx
function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return (
    // `AppField`は`Field`の代わりに使用します。`AppField`が必要なコンテキストを提供します
    <form.AppField
      name="firstName"
      children={(field) => <field.TextField label="First Name" />}
    />
  )
}
```

これにより、共有コンポーネントのUIを再利用できるだけでなく、TanStack Formから期待される型安全性も保持されます: `name`にタイポがあるとTypeScriptエラーが発生します。

#### パフォーマンスに関する注意

コンテキストはReactエコシステムで価値のあるツールですが、コンテキストを通じてリアクティブな値を提供すると不要な再レンダリングが発生するという懸念が多くのユーザーから提起されています。

> このパフォーマンス問題に不慣れですか？[Mark Eriksonのブログ記事（Reduxがこれらの問題の多くを解決する理由を説明）](https://blog.isquaredsoftware.com/2021/01/context-redux-differences/)が良い出発点です。

これは指摘すべき良い懸念ですが、TanStack Formにとっては問題ではありません。コンテキストを通じて提供される値自体はリアクティブではなく、代わりにリアクティブなプロパティを持つ静的なクラスインスタンスです（[TanStack Storeをシグナル実装として使用](https://tanstack.com/store)）。

### 事前バインドされたフォームコンポーネント

`form.AppField`はFieldのボイラープレートと再利用性に関する多くの問題を解決しますが、*フォーム*のボイラープレートと再利用性の問題は解決しません。

特に、リアクティブなフォーム送信ボタンなどのために`form.Subscribe`のインスタンスを共有できるようにすることは一般的なユースケースです。

```tsx
function SubscribeButton({ label }: { label: string }) {
  const form = useFormContext()
  return (
    <form.Subscribe selector={(state) => state.isSubmitting}>
      {(isSubmitting) => <button disabled={isSubmitting}>{label}</button>}
    </form.Subscribe>
  )
}

const { useAppForm, withForm } = createFormHook({
  fieldComponents: {},
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return (
    <form.AppForm>
      //
      `AppForm`コンポーネントラッパーに注意してください。`AppForm`が必要なコンテキストを提供します
      <form.SubscribeButton label="Submit" />
    </form.AppForm>
  )
}
```

## 大きなフォームを小さな部分に分割

時々フォームは非常に大きくなります。そういうこともあります。TanStack Formは大きなフォームをうまくサポートしていますが、何百行、何千行ものコードのファイルを扱うのは楽しいことではありません。

これを解決するために、`withForm`高階コンポーネントを使用してフォームを小さな部分に分割することをサポートしています。

```tsx
const { useAppForm, withForm } = createFormHook({
  fieldComponents: {
    TextField,
  },
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

const ChildForm = withForm({
  // これらの値は型チェックのみに使用され、実行時には使用されません
  // これにより、`formOptions`から`...formOpts`を使用でき、オプションを再宣言する必要がなくなります
  defaultValues: {
    firstName: 'John',
    lastName: 'Doe',
  },
  // オプションですが、`form`に加えて`render`関数にpropsを追加します
  props: {
    // これらのpropsも`render`関数のデフォルト値として設定されます
    title: 'Child Form',
  },
  render: function Render({ form, title }) {
    return (
      <div>
        <p>{title}</p>
        <form.AppField
          name="firstName"
          children={(field) => <field.TextField label="First Name" />}
        />
        <form.AppForm>
          <form.SubscribeButton label="Submit" />
        </form.AppForm>
      </div>
    )
  },
})

function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return <ChildForm form={form} title={'Testing'} />
}
```

### `withForm` FAQ

> なぜフックではなく高階コンポーネントなのですか？

フックはReactの未来ですが、高階コンポーネントは依然として構成のための強力なツールです。特に、`useForm`のAPIにより、ユーザーがジェネリックを渡す必要なく強力な型安全性を実現できます。

> `render`内でフックに関するESLintエラーが発生するのはなぜですか？

ESLintは関数のトップレベルでフックを探し、`render`がどのように定義されたかによってはトップレベルのコンポーネントとして認識されない場合があります。

```tsx
// これはフック使用時にESLintエラーを引き起こします
const ChildForm = withForm({
  // ...
  render: ({ form, title }) => {
    // ...
  },
})
```

```tsx
// これは問題なく動作します
const ChildForm = withForm({
  // ...
  render: function Render({ form, title }) {
    // ...
  },
})
```

## フォームとフィールドコンポーネントのツリーシェイキング

上記の例は始めるのに良いですが、何百ものフォームやフィールドコンポーネントがある特定のユースケースには理想的ではありません。
特に、フォームフックを使用するすべてのファイルのバンドルにすべてのフォームやフィールドコンポーネントを含めたくない場合があります。

これを解決するために、`createFormHook` TanStack APIとReactの`lazy`および`Suspense`コンポーネントを組み合わせることができます:

```typescript
// src/hooks/form-context.ts
import { createFormHookContexts } from '@tanstack/react-form'

export const { fieldContext, useFieldContext, formContext, useFormContext } =
  createFormHookContexts()
```

```tsx
// src/components/text-field.tsx
import { useFieldContext } from '../hooks/form-context.tsx'

export default function TextField({ label }: { label: string }) {
  const field = useFieldContext<string>()

  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}
```

```tsx
// src/hooks/form.ts
import { lazy } from 'react'
import { createFormHook } from '@tanstack/react-form'

const TextField = lazy(() => import('../components/text-fields.tsx'))

const { useAppForm, withForm } = createFormHook({
  fieldContext,
  formContext,
  fieldComponents: {
    TextField,
  },
  formComponents: {},
})
```

```tsx
// src/App.tsx
import { Suspense } from 'react'
import { PeoplePage } from './features/people/page.tsx'

export default function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <PeopleForm />
    </Suspense>
  )
}
```

これにより、`TextField`コンポーネントがロードされている間はSuspenseフォールバックが表示され、ロードが完了するとフォームがレンダリングされます。

## すべてをまとめる

カスタムフォームフックの作成の基本を説明したので、1つの例ですべてをまとめましょう。

```tsx
// /src/hooks/form.ts、アプリ全体で使用
const { fieldContext, useFieldContext, formContext, useFormContext } =
  createFormHookContexts()

function TextField({ label }: { label: string }) {
  const field = useFieldContext<string>()
  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}

function SubscribeButton({ label }: { label: string }) {
  const form = useFormContext()
  return (
    <form.Subscribe selector={(state) => state.isSubmitting}>
      {(isSubmitting) => <button disabled={isSubmitting}>{label}</button>}
    </form.Subscribe>
  )
}

const { useAppForm, withForm } = createFormHook({
  fieldComponents: {
    TextField,
  },
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

// /src/features/people/shared-form.ts、`people`機能全体で使用
const formOpts = formOptions({
  defaultValues: {
    firstName: 'John',
    lastName: 'Doe',
  },
})

// /src/features/people/nested-form.ts、`people`ページで使用
const ChildForm = withForm({
  ...formOpts,
  // オプションですが、`form`の外で`render`関数にpropsを追加します
  props: {
    title: 'Child Form',
  },
  render: ({ form, title }) => {
    return (
      <div>
        <p>{title}</p>
        <form.AppField
          name="firstName"
          children={(field) => <field.TextField label="First Name" />}
        />
        <form.AppForm>
          <form.SubscribeButton label="Submit" />
        </form.AppForm>
      </div>
    )
  },
})

// /src/features/people/page.ts
const Parent = () => {
  const form = useAppForm({
    ...formOpts,
  })

  return <ChildForm form={form} title={'Testing'} />
}
```

## API使用ガイダンス

使用すべきAPIを決定するのに役立つチャートを以下に示します:

![](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/react_form_composability.svg)
