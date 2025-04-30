---
source-updated-at: '2025-03-28T18:10:56.000Z'
translation-updated-at: '2025-04-30T22:07:18.195Z'
id: ssr
title: SSR/TanStack Start/Next.js
---

# استخدام إطار العمل الميتا (Meta-Framework) مع React

`TanStack Form` متوافق مع `React` بشكل مباشر، حيث يدعم `SSR` وهو مستقل عن إطار العمل. ومع ذلك، هناك تكوينات محددة مطلوبة حسب إطار العمل الذي تختاره.

اليوم نحن ندعم أطر العمل الميتا التالية:

- [TanStack Start](https://tanstack.com/start/)
- [Next.js](https://nextjs.org/)
- [Remix](https://remix.run)

## استخدام TanStack Form مع TanStack Start

هذا القسم يركز على دمج `TanStack Form` مع `TanStack Start`.

### متطلبات TanStack Start

- ابدأ مشروعًا جديدًا بـ `TanStack Start` باتباع الخطوات في [دليل البداية السريعة لـ TanStack Start](https://tanstack.com/router/latest/docs/framework/react/guide/tanstack-start)
- قم بتثبيت `@tanstack/react-form`

### بدء الدمج

لنبدأ بإنشاء `formOption` والذي سنستخدمه لمشاركة هيكل النموذج بين العميل والخادم.

```typescript
// app/routes/index.tsx, ولكن يمكن استخراجه إلى أي مسار آخر
import { formOptions } from '@tanstack/react-form'

// يمكنك تمرير خيارات النموذج الأخرى هنا
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

بعد ذلك، يمكننا إنشاء [إجراء خادم Start](https://tanstack.com/start/latest/docs/framework/react/server-functions) والذي سيتعامل مع إرسال النموذج على الخادم.

```typescript
// app/routes/index.tsx, ولكن يمكن استخراجه إلى أي مسار آخر
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
      // احفظ بيانات النموذج في قاعدة البيانات
      // await sql`
      //   INSERT INTO users (name, email, password)
      //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
      // `
    } catch (e) {
      if (e instanceof ServerValidateError) {
        // سجل أخطاء النموذج أو نفذ أي منطق آخر هنا
        return e.response
      }

      // حدث خطأ آخر أثناء تحليل النموذج
      console.error(e)
      setResponseStatus(500)
      return 'There was an internal error'
    }

    return 'Form has validated successfully'
  })
```

ثم نحتاج إلى إنشاء طريقة لاستخراج بيانات النموذج من `response` الخاص بـ `serverValidate` باستخدام إجراء خادم آخر:

```typescript
// app/routes/index.tsx, ولكن يمكن استخراجه إلى أي مسار آخر
import { getFormData } from '@tanstack/react-form/start'

export const getFormDataFromServer = createServerFn({ method: 'GET' }).handler(
  async () => {
    return getFormData()
  },
)
```

أخيرًا، سنستخدم `getFormDataFromServer` في محملنا (loader) لجلب الحالة من الخادم إلى العميل و `handleForm` في مكون النموذج على جانب العميل.

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

## استخدام TanStack Form مع Next.js App Router

> قبل قراءة هذا القسم، يُقترح أن تفهم كيفية عمل مكونات الخادم (Server Components) وإجراءات الخادم (Server Actions) في React. [اطلع على هذه السلسلة من المدونات لمزيد من المعلومات](https://playfulprogramming.com/collections/react-beyond-the-render)

هذا القسم يركز على دمج `TanStack Form` مع `Next.js`، خاصة باستخدام `App Router` و `Server Actions`.

### متطلبات Next.js

- ابدأ مشروعًا جديدًا بـ `Next.js` باتباع الخطوات في [توثيق Next.js](https://nextjs.org/docs/getting-started/installation). تأكد من اختيار `نعم` لـ `Would you like to use App Router?` أثناء الإعداد للوصول إلى جميع الميزات الجديدة التي يوفرها Next.js.
- قم بتثبيت `@tanstack/react-form`
- قم بتثبيت أي [مدقق للنماذج](/form/latest/docs/framework/react/guides/validation#validation-through-schema-libraries) تختاره. [اختياري]

## دمج App Router

لنبدأ بإنشاء `formOption` والذي سنستخدمه لمشاركة هيكل النموذج بين العميل والخادم.

```typescript
// shared-code.ts
// لاحظ أن مسار الاستيراد مختلف عن جانب العميل
import { formOptions } from '@tanstack/react-form/nextjs'

// يمكنك تمرير خيارات النموذج الأخرى هنا
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

بعد ذلك، يمكننا إنشاء [إجراء خادم React](https://playfulprogramming.com/posts/what-are-react-server-components) والذي سيتعامل مع إرسال النموذج على الخادم.

```typescript
// action.ts
'use server'

// لاحظ أن مسار الاستيراد مختلف عن جانب العميل
import {
  ServerValidateError,
  createServerValidate,
} from '@tanstack/react-form/nextjs'
import { formOpts } from './shared-code'

// أنشئ إجراء الخادم الذي سيستنتج أنواع النموذج من `formOpts`
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
    // احفظ بيانات النموذج في قاعدة البيانات
    // await sql`
    //   INSERT INTO users (name, email, password)
    //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
    // `
  } catch (e) {
    if (e instanceof ServerValidateError) {
      return e.formState
    }

    // حدث خطأ آخر أثناء التحقق من صحة النموذج
    throw e
  }

  // تم التحقق من صحة النموذج بنجاح!
}
```

أخيرًا، سنستخدم `someAction` في مكون النموذج على جانب العميل.

```tsx
// client-component.tsx
'use client'

import { useActionState } from 'react'
import { initialFormState } from '@tanstack/react-form/nextjs'
// لاحظ أن الاستيراد من `react-form` وليس `react-form/nextjs`
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

هنا، نستخدم [ربط React `useActionState`](https://playfulprogramming.com/posts/what-is-use-action-state-and-form-status) وربط `useTransform` من TanStack Form لدمج الحالة المعادة من إجراء الخادم مع حالة النموذج.

> إذا واجهت الخطأ التالي في تطبيق Next.js:
>
> ```typescript
> x You're importing a component that needs `useState`. This React hook only works in a client component. To fix, mark the file (or its parent) with the `"use client"` directive.
> ```
>
> هذا لأنك لا تستورد كود جانب الخادم من `@tanstack/react-form/nextjs`. تأكد من استيراد الوحدة الصحيحة بناءً على البيئة.
>
> [هذا قيد في Next.js](https://github.com/phryneas/rehackt). من المحتمل ألا تواجه أطر العمل الميتا الأخرى نفس المشكلة.

## استخدام TanStack Form مع Remix

> قبل قراءة هذا القسم، يُقترح أن تفهم كيفية عمل إجراءات Remix. [اطلع على توثيق Remix لمزيد من المعلومات](https://remix.run/docs/en/main/discussion/data-flow#route-action)

### متطلبات Remix

- ابدأ مشروعًا جديدًا بـ `Remix` باتباع الخطوات في [توثيق Remix](https://remix.run/docs/en/main/start/quickstart).
- قم بتثبيت `@tanstack/react-form`
- قم بتثبيت أي [مدقق للنماذج](/form/latest/docs/framework/react/guides/validation#validation-through-schema-libraries) تختاره. [اختياري]

## دمج Remix

لنبدأ بإنشاء `formOption` والذي سنستخدمه لمشاركة هيكل النموذج بين العميل والخادم.

```typescript
// routes/_index/route.tsx
import { formOptions } from '@tanstack/react-form/remix'

// يمكنك تمرير خيارات النموذج الأخرى هنا
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

بعد ذلك، يمكننا إنشاء [إجراء](https://remix.run/docs/en/main/discussion/data-flow#route-action) والذي سيتعامل مع إرسال النموذج على الخادم.

```tsx
// routes/_index/route.tsx

import {
  ServerValidateError,
  createServerValidate,
  formOptions,
} from '@tanstack/react-form/remix'

import type { ActionFunctionArgs } from '@remix-run/node'

// export const formOpts = formOptions({

// أنشئ إجراء الخادم الذي سيستنتج أنواع النموذج من `formOpts`
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
    // احفظ بيانات النموذج في قاعدة البيانات
    // await sql`
    //   INSERT INTO users (name, email, password)
    //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
    // `
  } catch (e) {
    if (e instanceof ServerValidateError) {
      return e.formState
    }

    // حدث خطأ آخر أثناء التحقق من صحة النموذج
    throw e
  }

  // تم التحقق من صحة النموذج بنجاح!
}
```

أخيرًا، سيتم استدعاء `action` عند إرسال النموذج.

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

هنا، نستخدم [ربط Remix `useActionData`](https://remix.run/docs/en/main/hooks/use-action-data) وربط `useTransform` من TanStack Form لدمج الحالة المعادة من إجراء الخادم مع حالة النموذج.
