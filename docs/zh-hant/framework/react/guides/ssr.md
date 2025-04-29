---
source-updated-at: '2025-03-28T18:10:56.000Z'
translation-updated-at: '2025-04-25T20:39:00.310Z'
id: ssr
title: 伺服器渲染 (SSR)/TanStack Start/Next.js
---

TanStack Form 預設即與 React 相容，支援 `SSR` 且為框架無關 (framework-agnostic)。但根據您選擇的框架，仍需進行特定配置。

目前我們支援以下元框架 (meta-frameworks)：

- [TanStack Start](https://tanstack.com/start/)
- [Next.js](https://nextjs.org/)
- [Remix](https://remix.run)

## 在 TanStack Start 中使用 TanStack Form

本節重點說明如何將 TanStack Form 整合至 TanStack Start。

### TanStack Start 前置準備

- 依照 [TanStack Start 快速入門指南](https://tanstack.com/router/latest/docs/framework/react/guide/tanstack-start) 建立新專案
- 安裝 `@tanstack/react-form`

### 開始整合

首先建立一個 `formOption`，用於在客戶端與伺服器端共享表單結構：

```typescript
// app/routes/index.tsx，也可提取至其他路徑
import { formOptions } from '@tanstack/react-form'

// 可在此傳遞其他表單選項
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

接著建立 [Start 伺服器動作 (Server Action)](https://tanstack.com/start/latest/docs/framework/react/server-functions) 來處理伺服器端的表單提交：

```typescript
// app/routes/index.tsx，也可提取至其他路徑
import {
  createServerValidate,
  ServerValidateError,
} from '@tanstack/react-form/start'

const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return '伺服器驗證：註冊年齡必須至少 12 歲'
    }
  },
})

export const handleForm = createServerFn({
  method: 'POST',
})
  .validator((data: unknown) => {
    if (!(data instanceof FormData)) {
      throw new Error('無效的表單資料')
    }
    return data
  })
  .handler(async (ctx) => {
    try {
      const validatedData = await serverValidate(ctx.data)
      console.log('驗證資料', validatedData)
      // 將表單資料存入資料庫
      // await sql`
      //   INSERT INTO users (name, email, password)
      //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
      // `
    } catch (e) {
      if (e instanceof ServerValidateError) {
        // 在此記錄表單錯誤或執行其他邏輯
        return e.response
      }

      // 解析表單時發生其他錯誤
      console.error(e)
      setResponseStatus(500)
      return '發生內部錯誤'
    }

    return '表單驗證成功'
  })
```

然後建立另一個伺服器動作來從 `serverValidate` 的 `response` 獲取表單資料：

```typescript
// app/routes/index.tsx，也可提取至其他路徑
import { getFormData } from '@tanstack/react-form/start'

export const getFormDataFromServer = createServerFn({ method: 'GET' }).handler(
  async () => {
    return getFormData()
  },
)
```

最後在 loader 中使用 `getFormDataFromServer` 將伺服器狀態傳遞至客戶端，並在客戶端表單元件中使用 `handleForm`：

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
            value < 8 ? '客戶端驗證：年齡必須至少 8 歲' : undefined,
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
            {isSubmitting ? '...' : '提交'}
          </button>
        )}
      </form.Subscribe>
    </form>
  )
}
```

## 在 Next.js App Router 中使用 TanStack Form

> 閱讀本節前，建議先了解 React 伺服器元件 (Server Components) 和伺服器動作 (Server Actions) 的運作原理。[參考此部落格系列獲取更多資訊](https://playfulprogramming.com/collections/react-beyond-the-render)

本節重點說明如何將 TanStack Form 整合至 `Next.js`，特別是使用 `App Router` 和 `Server Actions`。

### Next.js 前置準備

- 依照 [Next.js 文件](https://nextjs.org/docs/getting-started/installation) 建立新專案。設定時請對 `Would you like to use App Router?` 選擇 `yes` 以使用 Next.js 提供的新功能
- 安裝 `@tanstack/react-form`
- 安裝任選的 [表單驗證器](/form/latest/docs/framework/react/guides/validation#validation-through-schema-libraries) [選用]

## App Router 整合

首先建立一個 `formOption`，用於在客戶端與伺服器端共享表單結構：

```typescript
// shared-code.ts
// 注意導入路徑與客戶端不同
import { formOptions } from '@tanstack/react-form/nextjs'

// 可在此傳遞其他表單選項
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

接著建立 [React 伺服器動作](https://playfulprogramming.com/posts/what-are-react-server-components) 來處理伺服器端的表單提交：

```typescript
// action.ts
'use server'

// 注意導入路徑與客戶端不同
import {
  ServerValidateError,
  createServerValidate,
} from '@tanstack/react-form/nextjs'
import { formOpts } from './shared-code'

// 建立伺服器動作，將從 `formOpts` 推斷表單類型
const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return '伺服器驗證：註冊年齡必須至少 12 歲'
    }
  },
})

export default async function someAction(prev: unknown, formData: FormData) {
  try {
    const validatedData = await serverValidate(formData)
    console.log('驗證資料', validatedData)
    // 將表單資料存入資料庫
    // await sql`
    //   INSERT INTO users (name, email, password)
    //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
    // `
  } catch (e) {
    if (e instanceof ServerValidateError) {
      return e.formState
    }

    // 驗證表單時發生其他錯誤
    throw e
  }

  // 表單驗證成功！
}
```

最後在客戶端表單元件中使用 `someAction`：

```tsx
// client-component.tsx
'use client'

import { useActionState } from 'react'
import { initialFormState } from '@tanstack/react-form/nextjs'
// 注意從 `react-form` 而非 `react-form/nextjs` 導入
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
            value < 8 ? '客戶端驗證：年齡必須至少 8 歲' : undefined,
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
            {isSubmitting ? '...' : '提交'}
          </button>
        )}
      </form.Subscribe>
    </form>
  )
}
```

此處我們使用 [React 的 `useActionState` 鉤子](https://playfulprogramming.com/posts/what-is-use-action-state-and-form-status) 和 TanStack Form 的 `useTransform` 鉤子來合併從伺服器動作返回的狀態與表單狀態。

> 若在 Next.js 應用中遇到以下錯誤：
>
> ```typescript
> x 您正在導入需要 `useState` 的元件。此 React 鉤子僅能在客戶端元件中運作。解決方法：在檔案（或其父級）加上 `"use client"` 指令
> ```
>
> 這是因為您未從 `@tanstack/react-form/nextjs` 導入伺服器端程式碼。請確保根據環境導入正確的模組。
>
> [這是 Next.js 的限制](https://github.com/phryneas/rehackt)。其他元框架可能不會有相同問題。

## 在 Remix 中使用 TanStack Form

> 閱讀本節前，建議先了解 Remix 動作的運作原理。[參考 Remix 文件獲取更多資訊](https://remix.run/docs/en/main/discussion/data-flow#route-action)

### Remix 前置準備

- 依照 [Remix 文件](https://remix.run/docs/en/main/start/quickstart) 建立新專案
- 安裝 `@tanstack/react-form`
- 安裝任選的 [表單驗證器](/form/latest/docs/framework/react/guides/validation#validation-through-schema-libraries) [選用]

## Remix 整合

首先建立一個 `formOption`，用於在客戶端與伺服器端共享表單結構：

```typescript
// routes/_index/route.tsx
import { formOptions } from '@tanstack/react-form/remix'

// 可在此傳遞其他表單選項
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

接著建立 [動作 (action)](https://remix.run/docs/en/main/discussion/data-flow#route-action) 來處理伺服器端的表單提交：

```tsx
// routes/_index/route.tsx

import {
  ServerValidateError,
  createServerValidate,
  formOptions,
} from '@tanstack/react-form/remix'

import type { ActionFunctionArgs } from '@remix-run/node'

// export const formOpts = formOptions({

// 建立伺服器動作，將從 `formOpts` 推斷表單類型
const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return '伺服器驗證：註冊年齡必須至少 12 歲'
    }
  },
})

export async function action({ request }: ActionFunctionArgs) {
  const formData = await request.formData()
  try {
    const validatedData = await serverValidate(formData)
    console.log('驗證資料', validatedData)
    // 將表單資料存入資料庫
    // await sql`
    //   INSERT INTO users (name, email, password)
    //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
    // `
  } catch (e) {
    if (e instanceof ServerValidateError) {
      return e.formState
    }

    // 驗證表單時發生其他錯誤
    throw e
  }

  // 表單驗證成功！
}
```

最後，提交表單時會呼叫 `action`：

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
            value < 8 ? '客戶端驗證：年齡必須至少 8 歲' : undefined,
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
            {isSubmitting ? '...' : '提交'}
          </button>
        )}
      </form.Subscribe>
    </Form>
  )
}
```

此處我們使用 [Remix 的 `useActionData` 鉤子](https://remix.run/docs/en/main/hooks/use-action-data) 和 TanStack Form 的 `useTransform` 鉤子來合併從伺服器動作返回的狀態與表單狀態。
