---
source-updated-at: '2025-03-28T18:10:56.000Z'
translation-updated-at: '2025-04-30T22:01:38.906Z'
id: ssr
title: SSR/TanStack Start/Next.js
---

# Использование TanStack Form с мета-фреймворками React

TanStack Form совместим с React из коробки, поддерживает `SSR` и является независимым от фреймворков. Однако требуются специфические настройки в зависимости от выбранного вами фреймворка.

На данный момент поддерживаются следующие мета-фреймворки:

- [TanStack Start](https://tanstack.com/start/)
- [Next.js](https://nextjs.org/)
- [Remix](https://remix.run)

## Использование TanStack Form в TanStack Start

Этот раздел посвящен интеграции TanStack Form с TanStack Start.

### Предварительные требования для TanStack Start

- Создайте новый проект `TanStack Start`, следуя шагам из [Руководства по быстрому старту TanStack Start](https://tanstack.com/router/latest/docs/framework/react/guide/tanstack-start)
- Установите `@tanstack/react-form`

### Интеграция

Начнем с создания `formOption`, который мы будем использовать для передачи структуры формы между клиентом и сервером.

```typescript
// app/routes/index.tsx, но можно вынести в любой другой путь
import { formOptions } from '@tanstack/react-form'

// Здесь можно передать другие параметры формы
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

Затем создадим [Server Action в Start](https://tanstack.com/start/latest/docs/framework/react/server-functions), которая будет обрабатывать отправку формы на сервере.

```typescript
// app/routes/index.tsx, но можно вынести в любой другой путь
import {
  createServerValidate,
  ServerValidateError,
} from '@tanstack/react-form/start'

const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return 'Server validation: You must be at least 12 to sign up'
    }
  },
})

export const handleForm = createServerFn({
  method: 'POST',
})
  .validator((data: unknown) => {
    if (!(data instanceof FormData)) {
      throw new Error('Invalid form data')
    }
    return data
  })
  .handler(async (ctx) => {
    try {
      const validatedData = await serverValidate(ctx.data)
      console.log('validatedData', validatedData)
      // Сохраняем данные формы в базу данных
      // await sql`
      //   INSERT INTO users (name, email, password)
      //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
      // `
    } catch (e) {
      if (e instanceof ServerValidateError) {
        // Логируем ошибки формы или выполняем другую логику
        return e.response
      }

      // Произошла другая ошибка при обработке формы
      console.error(e)
      setResponseStatus(500)
      return 'There was an internal error'
    }

    return 'Form has validated successfully'
  })
```

Затем нам нужно создать способ получения данных формы из `response` `serverValidate` с помощью другой server action:

```typescript
// app/routes/index.tsx, но можно вынести в любой другой путь
import { getFormData } from '@tanstack/react-form/start'

export const getFormDataFromServer = createServerFn({ method: 'GET' }).handler(
  async () => {
    return getFormData()
  },
)
```

Наконец, мы используем `getFormDataFromServer` в нашем загрузчике (loader), чтобы передать состояние с сервера на клиент, и `handleForm` в клиентском компоненте формы.

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
            value < 8 ? 'Client validation: You must be at least 8' : undefined,
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
            {isSubmitting ? '...' : 'Submit'}
          </button>
        )}
      </form.Subscribe>
    </form>
  )
}
```

## Использование TanStack Form в Next.js App Router

> Перед прочтением этого раздела рекомендуется понять, как работают React Server Components и React Server Actions. [Подробнее в этой серии статей](https://playfulprogramming.com/collections/react-beyond-the-render)

Этот раздел посвящен интеграции TanStack Form с `Next.js`, в частности с использованием `App Router` и `Server Actions`.

### Предварительные требования для Next.js

- Создайте новый проект `Next.js`, следуя шагам из [документации Next.js](https://nextjs.org/docs/getting-started/installation). Убедитесь, что вы выбрали `yes` для `Would you like to use App Router?` во время настройки, чтобы получить доступ ко всем новым функциям Next.js.
- Установите `@tanstack/react-form`
- Установите любой [валидатор форм](/form/latest/docs/framework/react/guides/validation#validation-through-schema-libraries) по вашему выбору. [Опционально]

## Интеграция с App Router

Начнем с создания `formOption`, который мы будем использовать для передачи структуры формы между клиентом и сервером.

```typescript
// shared-code.ts
// Обратите внимание, что путь импорта отличается от клиентского
import { formOptions } from '@tanstack/react-form/nextjs'

// Здесь можно передать другие параметры формы
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

Затем создадим [React Server Action](https://playfulprogramming.com/posts/what-are-react-server-components), которая будет обрабатывать отправку формы на сервере.

```typescript
// action.ts
'use server'

// Обратите внимание, что путь импорта отличается от клиентского
import {
  ServerValidateError,
  createServerValidate,
} from '@tanstack/react-form/nextjs'
import { formOpts } from './shared-code'

// Создаем server action, которая будет выводить типы формы из `formOpts`
const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return 'Server validation: You must be at least 12 to sign up'
    }
  },
})

export default async function someAction(prev: unknown, formData: FormData) {
  try {
    const validatedData = await serverValidate(formData)
    console.log('validatedData', validatedData)
    // Сохраняем данные формы в базу данных
    // await sql`
    //   INSERT INTO users (name, email, password)
    //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
    // `
  } catch (e) {
    if (e instanceof ServerValidateError) {
      return e.formState
    }

    // Произошла другая ошибка при валидации формы
    throw e
  }

  // Ваша форма успешно прошла валидацию!
}
```

Наконец, мы используем `someAction` в нашем клиентском компоненте формы.

```tsx
// client-component.tsx
'use client'

import { useActionState } from 'react'
import { initialFormState } from '@tanstack/react-form/nextjs'
// Обратите внимание, что импорт идет из `react-form`, а не `react-form/nextjs`
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
            value < 8 ? 'Client validation: You must be at least 8' : undefined,
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
            {isSubmitting ? '...' : 'Submit'}
          </button>
        )}
      </form.Subscribe>
    </form>
  )
}
```

Здесь мы используем [хук `useActionState` из React](https://playfulprogramming.com/posts/what-is-use-action-state-and-form-status) и хук `useTransform` из TanStack Form для объединения состояния, возвращаемого server action, с состоянием формы.

> Если вы получите следующую ошибку в вашем приложении Next.js:
>
> ```typescript
> x You're importing a component that needs `useState`. This React hook only works in a client component. To fix, mark the file (or its parent) with the `"use client"` directive.
> ```
>
> Это происходит потому, что вы импортируете серверный код не из `@tanstack/react-form/nextjs`. Убедитесь, что вы импортируете правильный модуль в зависимости от окружения.
>
> [Это ограничение Next.js](https://github.com/phryneas/rehackt). Другие мета-фреймворки, скорее всего, не будут иметь такой же проблемы.

## Использование TanStack Form в Remix

> Перед прочтением этого раздела рекомендуется понять, как работают actions в Remix. [Подробнее в документации Remix](https://remix.run/docs/en/main/discussion/data-flow#route-action)

### Предварительные требования для Remix

- Создайте новый проект `Remix`, следуя шагам из [документации Remix](https://remix.run/docs/en/main/start/quickstart).
- Установите `@tanstack/react-form`
- Установите любой [валидатор форм](/form/latest/docs/framework/react/guides/validation#validation-through-schema-libraries) по вашему выбору. [Опционально]

## Интеграция с Remix

Начнем с создания `formOption`, который мы будем использовать для передачи структуры формы между клиентом и сервером.

```typescript
// routes/_index/route.tsx
import { formOptions } from '@tanstack/react-form/remix'

// Здесь можно передать другие параметры формы
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

Затем создадим [action](https://remix.run/docs/en/main/discussion/data-flow#route-action), который будет обрабатывать отправку формы на сервере.

```tsx
// routes/_index/route.tsx

import {
  ServerValidateError,
  createServerValidate,
  formOptions,
} from '@tanstack/react-form/remix'

import type { ActionFunctionArgs } from '@remix-run/node'

// export const formOpts = formOptions({

// Создаем server action, которая будет выводить типы формы из `formOpts`
const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return 'Server validation: You must be at least 12 to sign up'
    }
  },
})

export async function action({ request }: ActionFunctionArgs) {
  const formData = await request.formData()
  try {
    const validatedData = await serverValidate(formData)
    console.log('validatedData', validatedData)
    // Сохраняем данные формы в базу данных
    // await sql`
    //   INSERT INTO users (name, email, password)
    //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
    // `
  } catch (e) {
    if (e instanceof ServerValidateError) {
      return e.formState
    }

    // Произошла другая ошибка при валидации формы
    throw e
  }

  // Ваша форма успешно прошла валидацию!
}
```

Наконец, `action` будет вызван при отправке формы.

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
            value < 8 ? 'Client validation: You must be at least 8' : undefined,
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
            {isSubmitting ? '...' : 'Submit'}
          </button>
        )}
      </form.Subscribe>
    </Form>
  )
}
```

Здесь мы используем [хук `useActionData` из Remix](https://remix.run/docs/en/main/hooks/use-action-data) и хук `useTransform` из TanStack Form для объединения состояния, возвращаемого action, с состоянием формы.
