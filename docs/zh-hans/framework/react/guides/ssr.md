---
source-updated-at: '2025-03-28T18:10:56.000Z'
translation-updated-at: '2025-04-12T04:11:11.947Z'
id: ssr
title: SSR/TanStack Start/Next.js
---
TanStack Form 开箱即支持 React，兼容 服务端渲染 (SSR) 并保持框架无关性。但需要根据您选择的元框架进行特定配置。

目前我们支持以下元框架：

- [TanStack Start](https://tanstack.com/start/)
- [Next.js](https://nextjs.org/)
- [Remix](https://remix.run)

## 在 TanStack Start 中使用 TanStack Form

本节重点介绍如何将 TanStack Form 集成到 TanStack Start 中。

### TanStack Start 前置条件

- 按照 [TanStack Start 快速入门指南](https://tanstack.com/router/latest/docs/framework/react/guide/tanstack-start) 新建项目
- 安装 `@tanstack/react-form`

### 开始集成

首先创建用于在客户端和服务端共享表单结构的 `formOption`：

```typescript
// app/routes/index.tsx（可提取到其他路径）
import { formOptions } from '@tanstack/react-form'

// 可在此传入其他表单配置
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

接着创建 [Start 服务端动作](https://tanstack.com/start/latest/docs/framework/react/server-functions) 来处理服务端表单提交：

```typescript
// app/routes/index.tsx（可提取到其他路径）
import {
  createServerValidate,
  ServerValidateError,
} from '@tanstack/react-form/start'

const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return '服务端验证：注册年龄必须至少12岁'
    }
  },
})

export const handleForm = createServerFn({
  method: 'POST',
})
  .validator((data: unknown) => {
    if (!(data instanceof FormData)) {
      throw new Error('无效的表单数据')
    }
    return data
  })
  .handler(async (ctx) => {
    try {
      const validatedData = await serverValidate(ctx.data)
      console.log('验证数据', validatedData)
      // 将表单数据持久化到数据库
      // await sql`
      //   INSERT INTO users (name, email, password)
      //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
      // `
    } catch (e) {
      if (e instanceof ServerValidateError) {
        // 在此记录表单错误或执行其他逻辑
        return e.response
      }

      // 解析表单时发生其他错误
      console.error(e)
      setResponseStatus(500)
      return '发生内部错误'
    }

    return '表单验证成功'
  })
```

然后通过另一个服务端动作获取 `serverValidate` 的响应数据：

```typescript
// app/routes/index.tsx（可提取到其他路径）
import { getFormData } from '@tanstack/react-form/start'

export const getFormDataFromServer = createServerFn({ method: 'GET' }).handler(
  async () => {
    return getFormData()
  },
)
```

最后在加载器中使用 `getFormDataFromServer` 将服务端状态传递到客户端，并在客户端表单组件中使用 `handleForm`：

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
            value < 8 ? '客户端验证：年龄必须至少8岁' : undefined,
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
            {isSubmitting ? '提交中...' : '提交'}
          </button>
        )}
      </form.Subscribe>
    </form>
  )
}
```

## 在 Next.js App Router 中使用 TanStack Form

> 阅读本节前，建议先了解 React 服务端组件 (Server Components) 和服务端动作 (Server Actions) 的工作原理。[参考此博客系列](https://playfulprogramming.com/collections/react-beyond-the-render)

本节重点介绍如何将 TanStack Form 集成到 Next.js 的 App Router 和服务端动作中。

### Next.js 前置条件

- 按照 [Next.js 文档](https://nextjs.org/docs/getting-started/installation) 新建项目，在设置过程中选择使用 App Router 以启用所有新特性
- 安装 `@tanstack/react-form`
- 安装任意 [表单验证库](/form/latest/docs/framework/react/guides/validation#validation-through-schema-libraries) [可选]

## App Router 集成

首先创建用于在客户端和服务端共享表单结构的 `formOption`：

```typescript
// shared-code.ts
// 注意导入路径与客户端不同
import { formOptions } from '@tanstack/react-form/nextjs'

// 可在此传入其他表单配置
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

接着创建 [React 服务端动作](https://playfulprogramming.com/posts/what-are-react-server-components) 来处理服务端表单提交：

```typescript
// action.ts
'use server'

// 注意导入路径与客户端不同
import {
  ServerValidateError,
  createServerValidate,
} from '@tanstack/react-form/nextjs'
import { formOpts } from './shared-code'

// 创建会从 formOpts 推断表单类型的服务端动作
const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return '服务端验证：注册年龄必须至少12岁'
    }
  },
})

export default async function someAction(prev: unknown, formData: FormData) {
  try {
    const validatedData = await serverValidate(formData)
    console.log('验证数据', validatedData)
    // 将表单数据持久化到数据库
    // await sql`
    //   INSERT INTO users (name, email, password)
    //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
    // `
  } catch (e) {
    if (e instanceof ServerValidateError) {
      return e.formState
    }

    // 验证表单时发生其他错误
    throw e
  }

  // 表单验证成功！
}
```

最后在客户端表单组件中使用 `someAction`：

```tsx
// client-component.tsx
'use client'

import { useActionState } from 'react'
import { initialFormState } from '@tanstack/react-form/nextjs'
// 注意从 react-form 而非 react-form/nextjs 导入
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
            value < 8 ? '客户端验证：年龄必须至少8岁' : undefined,
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
            {isSubmitting ? '提交中...' : '提交'}
          </button>
        )}
      </form.Subscribe>
    </form>
  )
}
```

此处我们结合使用了 [React 的 useActionState 钩子](https://playfulprogramming.com/posts/what-is-use-action-state-and-form-status) 和 TanStack Form 的 useTransform 钩子来合并服务端动作返回的状态与表单状态。

> 如果在 Next.js 应用中遇到以下错误：
> 
> ```typescript
> x 您正在导入需要 useState 的组件。此 React 钩子仅在客户端组件中有效。解决方法是在文件（或其父级）中添加 "use client" 指令
> ```
> 
> 这是因为您没有从 `@tanstack/react-form/nextjs` 导入服务端代码。请确保根据运行环境导入正确的模块。
> 
> [这是 Next.js 的已知限制](https://github.com/phryneas/rehackt)。其他元框架通常不会出现此问题。

## 在 Remix 中使用 TanStack Form

> 阅读本节前，建议先了解 Remix 动作的工作原理。[参考 Remix 文档](https://remix.run/docs/en/main/discussion/data-flow#route-action)

### Remix 前置条件

- 按照 [Remix 文档](https://remix.run/docs/en/main/start/quickstart) 新建项目
- 安装 `@tanstack/react-form`
- 安装任意 [表单验证库](/form/latest/docs/framework/react/guides/validation#validation-through-schema-libraries) [可选]

## Remix 集成

首先创建用于在客户端和服务端共享表单结构的 `formOption`：

```typescript
// routes/_index/route.tsx
import { formOptions } from '@tanstack/react-form/remix'

// 可在此传入其他表单配置
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

接着创建 [Remix 动作](https://remix.run/docs/en/main/discussion/data-flow#route-action) 来处理服务端表单提交：

```tsx
// routes/_index/route.tsx

import {
  ServerValidateError,
  createServerValidate,
  formOptions,
} from '@tanstack/react-form/remix'

import type { ActionFunctionArgs } from '@remix-run/node'

// export const formOpts = formOptions({

// 创建会从 formOpts 推断表单类型的服务端动作
const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return '服务端验证：注册年龄必须至少12岁'
    }
  },
})

export async function action({ request }: ActionFunctionArgs) {
  const formData = await request.formData()
  try {
    const validatedData = await serverValidate(formData)
    console.log('验证数据', validatedData)
    // 将表单数据持久化到数据库
    // await sql`
    //   INSERT INTO users (name, email, password)
    //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
    // `
  } catch (e) {
    if (e instanceof ServerValidateError) {
      return e.formState
    }

    // 验证表单时发生其他错误
    throw e
  }

  // 表单验证成功！
}
```

最后在表单提交时调用 `action`：

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
            value < 8 ? '客户端验证：年龄必须至少8岁' : undefined,
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
            {isSubmitting ? '提交中...' : '提交'}
          </button>
        )}
      </form.Subscribe>
    </Form>
  )
}
```

此处我们结合使用了 [Remix 的 useActionData 钩子](https://remix.run/docs/en/main/hooks/use-action-data) 和 TanStack Form 的 useTransform 钩子来合并服务端动作返回的状态与表单状态。
