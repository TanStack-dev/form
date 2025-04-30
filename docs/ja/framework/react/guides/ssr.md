---
source-updated-at: '2025-03-28T18:10:56.000Z'
translation-updated-at: '2025-04-30T21:27:22.310Z'
id: ssr
title: SSR/TanStack Start/Next.js
---

TanStack FormはReactとすぐに互換性があり、`SSR`をサポートし、フレームワークに依存しません。ただし、選択したフレームワークに応じて特定の設定が必要です。

現在、以下のメタフレームワークをサポートしています:

- [TanStack Start](https://tanstack.com/start/)
- [Next.js](https://nextjs.org/)
- [Remix](https://remix.run)

## TanStack StartでのTanStack Formの使用

このセクションでは、TanStack FormとTanStack Startの統合に焦点を当てます。

### TanStack Startの前提条件

- [TanStack Startクイックスタートガイド](https://tanstack.com/router/latest/docs/framework/react/guide/tanstack-start)の手順に従って新しい`TanStack Start`プロジェクトを開始
- `@tanstack/react-form`をインストール

### 統合の開始

まず、クライアントとサーバー間でフォームの形状を共有するために使用する`formOption`を作成します。

```typescript
// app/routes/index.tsxですが、他のパスに抽出することも可能
import { formOptions } from '@tanstack/react-form'

// ここで他のフォームオプションを渡すことができます
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

次に、サーバー上でフォーム送信を処理する[Start Server Action](https://tanstack.com/start/latest/docs/framework/react/server-functions)を作成します。

```typescript
// app/routes/index.tsxですが、他のパスに抽出することも可能
import {
  createServerValidate,
  ServerValidateError,
} from '@tanstack/react-form/start'

const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return 'サーバー検証: 登録には少なくとも12歳以上である必要があります'
    }
  },
})

export const handleForm = createServerFn({
  method: 'POST',
})
  .validator((data: unknown) => {
    if (!(data instanceof FormData)) {
      throw new Error('無効なフォームデータ')
    }
    return data
  })
  .handler(async (ctx) => {
    try {
      const validatedData = await serverValidate(ctx.data)
      console.log('validatedData', validatedData)
      // フォームデータをデータベースに保存
      // await sql`
      //   INSERT INTO users (name, email, password)
      //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
      // `
    } catch (e) {
      if (e instanceof ServerValidateError) {
        // フォームエラーを記録または他のロジックを実行
        return e.response
      }

      // フォーム解析中に他のエラーが発生
      console.error(e)
      setResponseStatus(500)
      return '内部エラーが発生しました'
    }

    return 'フォームの検証に成功しました'
  })
```

次に、別のサーバーアクションを使用して`serverValidate`の`response`からフォームデータを取得する方法を確立します:

```typescript
// app/routes/index.tsxですが、他のパスに抽出することも可能
import { getFormData } from '@tanstack/react-form/start'

export const getFormDataFromServer = createServerFn({ method: 'GET' }).handler(
  async () => {
    return getFormData()
  },
)
```

最後に、ローダーで`getFormDataFromServer`を使用してサーバーからクライアントに状態を取得し、クライアントサイドのフォームコンポーネントで`handleForm`を使用します。

```tsx
// app/routes/index.tsx
import { createFileRoute } from '@tanstack/react-router'
import {
  mergeForm,
  useForm,
  useStore,
  useTransform,
} from '@tanstack/react-form'

export const Route = createFileRoute('/')({
  component: Home,
  loader: async () => ({
    state: await getFormDataFromServer(),
  }),
})

function Home() {
  const { state } = Route.useLoaderData()
  const form = useForm({
    ...formOpts,
    transform: useTransform((baseForm) => mergeForm(baseForm, state), [state]),
  })

  const formErrors = useStore(form.store, (formState) => formState.errors)

  return (
    <form action={handleForm.url} method="post" encType={'multipart/form-data'}>
      {formErrors.map((error) => (
        <p key={error as string}>{error}</p>
      ))}

      <form.Field
        name="age"
        validators={{
          onChange: ({ value }) =>
            value < 8
              ? 'クライアント検証: 少なくとも8歳以上である必要があります'
              : undefined,
        }}
      >
        {(field) => {
          return (
            <div>
              <input
                name="age"
                type="number"
                value={field.state.value}
                onChange={(e) => field.handleChange(e.target.valueAsNumber)}
              />
              {field.state.meta.errors.map((error) => (
                <p key={error as string}>{error}</p>
              ))}
            </div>
          )
        }}
      </form.Field>
      <form.Subscribe
        selector={(formState) => [formState.canSubmit, formState.isSubmitting]}
      >
        {([canSubmit, isSubmitting]) => (
          <button type="submit" disabled={!canSubmit}>
            {isSubmitting ? '...' : '送信'}
          </button>
        )}
      </form.Subscribe>
    </form>
  )
}
```

## Next.js App RouterでのTanStack Formの使用

> このセクションを読む前に、React Server ComponentsとReact Server Actionsの動作を理解することをお勧めします。[詳細についてはこのブログシリーズを確認してください](https://playfulprogramming.com/collections/react-beyond-the-render)

このセクションでは、`Next.js`、特に`App Router`と`Server Actions`を使用したTanStack Formの統合に焦点を当てます。

### Next.jsの前提条件

- [Next.jsドキュメント](https://nextjs.org/docs/getting-started/installation)の手順に従って新しい`Next.js`プロジェクトを開始。セットアップ中に`App Routerを使用しますか？`で`yes`を選択し、Next.jsが提供するすべての新機能にアクセスできるようにします。
- `@tanstack/react-form`をインストール
- 任意の[フォームバリデータ](/form/latest/docs/framework/react/guides/validation#validation-through-schema-libraries)をインストール。[オプション]

## App Routerの統合

まず、クライアントとサーバー間でフォームの形状を共有するために使用する`formOption`を作成します。

```typescript
// shared-code.ts
// クライアントとは異なるインポートパスに注意
import { formOptions } from '@tanstack/react-form/nextjs'

// ここで他のフォームオプションを渡すことができます
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

次に、サーバー上でフォーム送信を処理する[React Server Action](https://playfulprogramming.com/posts/what-are-react-server-components)を作成します。

```typescript
// action.ts
'use server'

// クライアントとは異なるインポートパスに注意
import {
  ServerValidateError,
  createServerValidate,
} from '@tanstack/react-form/nextjs'
import { formOpts } from './shared-code'

// `formOpts`からフォームの型を推論するサーバーアクションを作成
const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return 'サーバー検証: 登録には少なくとも12歳以上である必要があります'
    }
  },
})

export default async function someAction(prev: unknown, formData: FormData) {
  try {
    const validatedData = await serverValidate(formData)
    console.log('validatedData', validatedData)
    // フォームデータをデータベースに保存
    // await sql`
    //   INSERT INTO users (name, email, password)
    //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
    // `
  } catch (e) {
    if (e instanceof ServerValidateError) {
      return e.formState
    }

    // フォーム検証中に他のエラーが発生
    throw e
  }

  // フォームの検証に成功しました!
}
```

最後に、クライアントサイドのフォームコンポーネントで`someAction`を使用します。

```tsx
// client-component.tsx
'use client'

import { useActionState } from 'react'
import { initialFormState } from '@tanstack/react-form/nextjs'
// `react-form/nextjs`ではなく`react-form`からのインポートに注意
import {
  mergeForm,
  useForm,
  useStore,
  useTransform,
} from '@tanstack/react-form'
import someAction from './action'
import { formOpts } from './shared-code'

export const ClientComp = () => {
  const [state, action] = useActionState(someAction, initialFormState)

  const form = useForm({
    ...formOpts,
    transform: useTransform((baseForm) => mergeForm(baseForm, state!), [state]),
  })

  const formErrors = useStore(form.store, (formState) => formState.errors)

  return (
    <form action={action as never} onSubmit={() => form.handleSubmit()}>
      {formErrors.map((error) => (
        <p key={error as string}>{error}</p>
      ))}

      <form.Field
        name="age"
        validators={{
          onChange: ({ value }) =>
            value < 8
              ? 'クライアント検証: 少なくとも8歳以上である必要があります'
              : undefined,
        }}
      >
        {(field) => {
          return (
            <div>
              <input
                name="age"
                type="number"
                value={field.state.value}
                onChange={(e) => field.handleChange(e.target.valueAsNumber)}
              />
              {field.state.meta.errors.map((error) => (
                <p key={error as string}>{error}</p>
              ))}
            </div>
          )
        }}
      </form.Field>
      <form.Subscribe
        selector={(formState) => [formState.canSubmit, formState.isSubmitting]}
      >
        {([canSubmit, isSubmitting]) => (
          <button type="submit" disabled={!canSubmit}>
            {isSubmitting ? '...' : '送信'}
          </button>
        )}
      </form.Subscribe>
    </form>
  )
}
```

ここでは、[Reactの`useActionState`フック](https://playfulprogramming.com/posts/what-is-use-action-state-and-form-status)とTanStack Formの`useTransform`フックを使用して、サーバーアクションから返された状態をフォーム状態とマージしています。

> Next.jsアプリケーションで次のエラーが発生した場合:
>
> ```typescript
> x `useState`が必要なコンポーネントをインポートしています。このReactフックはクライアントコンポーネントでのみ機能します。修正するには、ファイル（またはその親）に`"use client"`ディレクティブを追加してください。
> ```
>
> これは、`@tanstack/react-form/nextjs`からサーバーサイドコードをインポートしていないためです。環境に基づいて正しいモジュールをインポートしていることを確認してください。
>
> [これはNext.jsの制限です](https://github.com/phryneas/rehackt)。他のメタフレームワークでは同じ問題が発生しない可能性があります。

## RemixでのTanStack Formの使用

> このセクションを読む前に、Remixアクションの動作を理解することをお勧めします。[詳細についてはRemixのドキュメントを確認してください](https://remix.run/docs/en/main/discussion/data-flow#route-action)

### Remixの前提条件

- [Remixドキュメント](https://remix.run/docs/en/main/start/quickstart)の手順に従って新しい`Remix`プロジェクトを開始。
- `@tanstack/react-form`をインストール
- 任意の[フォームバリデータ](/form/latest/docs/framework/react/guides/validation#validation-through-schema-libraries)をインストール。[オプション]

## Remixの統合

まず、クライアントとサーバー間でフォームの形状を共有するために使用する`formOption`を作成します。

```typescript
// routes/_index/route.tsx
import { formOptions } from '@tanstack/react-form/remix'

// ここで他のフォームオプションを渡すことができます
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

次に、サーバー上でフォーム送信を処理する[アクション](https://remix.run/docs/en/main/discussion/data-flow#route-action)を作成します。

```tsx
// routes/_index/route.tsx

import {
  ServerValidateError,
  createServerValidate,
  formOptions,
} from '@tanstack/react-form/remix'

import type { ActionFunctionArgs } from '@remix-run/node'

// export const formOpts = formOptions({

// `formOpts`からフォームの型を推論するサーバーアクションを作成
const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return 'サーバー検証: 登録には少なくとも12歳以上である必要があります'
    }
  },
})

export async function action({ request }: ActionFunctionArgs) {
  const formData = await request.formData()
  try {
    const validatedData = await serverValidate(formData)
    console.log('validatedData', validatedData)
    // フォームデータをデータベースに保存
    // await sql`
    //   INSERT INTO users (name, email, password)
    //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
    // `
  } catch (e) {
    if (e instanceof ServerValidateError) {
      return e.formState
    }

    // フォーム検証中に他のエラーが発生
    throw e
  }

  // フォームの検証に成功しました!
}
```

最後に、フォーム送信時に`action`が呼び出されます。

```tsx
// routes/_index/route.tsx
import { Form, useActionData } from '@remix-run/react'

import {
  mergeForm,
  useForm,
  useStore,
  useTransform,
} from '@tanstack/react-form'
import {
  ServerValidateError,
  createServerValidate,
  formOptions,
  initialFormState,
} from '@tanstack/react-form/remix'

import type { ActionFunctionArgs } from '@remix-run/node'

// export const formOpts = formOptions({

// const serverValidate = createServerValidate({

// export async function action({request}: ActionFunctionArgs) {

export default function Index() {
  const actionData = useActionData<typeof action>()

  const form = useForm({
    ...formOpts,
    transform: useTransform(
      (baseForm) => mergeForm(baseForm, actionData ?? initialFormState),
      [actionData],
    ),
  })

  const formErrors = useStore(form.store, (formState) => formState.errors)

  return (
    <Form method="post" onSubmit={() => form.handleSubmit()}>
      {formErrors.map((error) => (
        <p key={error as string}>{error}</p>
      ))}

      <form.Field
        name="age"
        validators={{
          onChange: ({ value }) =>
            value < 8
              ? 'クライアント検証: 少なくとも8歳以上である必要があります'
              : undefined,
        }}
      >
        {(field) => {
          return (
            <div>
              <input
                name="age"
                type="number"
                value={field.state.value}
                onChange={(e) => field.handleChange(e.target.valueAsNumber)}
              />
              {field.state.meta.errors.map((error) => (
                <p key={error as string}>{error}</p>
              ))}
            </div>
          )
        }}
      </form.Field>
      <form.Subscribe
        selector={(formState) => [formState.canSubmit, formState.isSubmitting]}
      >
        {([canSubmit, isSubmitting]) => (
          <button type="submit" disabled={!canSubmit}>
            {isSubmitting ? '...' : '送信'}
          </button>
        )}
      </form.Subscribe>
    </Form>
  )
}
```

ここでは、[Remixの`useActionData`フック](https://remix.run/docs/en/main/hooks/use-action-data)とTanStack Formの`useTransform`フックを使用して、サーバーアクションから返された状態をフォーム状態とマージしています。
