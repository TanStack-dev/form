---
source-updated-at: '2025-03-28T18:10:56.000Z'
translation-updated-at: '2025-04-30T21:49:58.630Z'
id: ssr
title: SSR/TanStack Start/Next.js
---

TanStack Form es compatible con React de forma predeterminada, soportando `SSR` y siendo independiente del framework. Sin embargo, se requieren configuraciones específicas según el framework elegido.

Actualmente, admitimos los siguientes meta-frameworks:

- [TanStack Start](https://tanstack.com/start/)
- [Next.js](https://nextjs.org/)
- [Remix](https://remix.run)

## Uso de TanStack Form en TanStack Start

Esta sección se centra en la integración de TanStack Form con TanStack Start.

### Requisitos previos de TanStack Start

- Inicie un nuevo proyecto `TanStack Start`, siguiendo los pasos de la [Guía de inicio rápido de TanStack Start](https://tanstack.com/router/latest/docs/framework/react/guide/tanstack-start)
- Instale `@tanstack/react-form`

### Integración inicial

Comencemos creando un `formOption` que usaremos para compartir la estructura del formulario entre el cliente y el servidor.

```typescript
// app/routes/index.tsx, pero puede extraerse a cualquier otra ruta
import { formOptions } from '@tanstack/react-form'

// Puede pasar otras opciones de formulario aquí
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

A continuación, podemos crear [una Acción de Servidor de Start](https://tanstack.com/start/latest/docs/framework/react/server-functions) que manejará el envío del formulario en el servidor.

```typescript
// app/routes/index.tsx, pero puede extraerse a cualquier otra ruta
import {
  createServerValidate,
  ServerValidateError,
} from '@tanstack/react-form/start'

const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return 'Validación del servidor: Debes tener al menos 12 años para registrarte'
    }
  },
})

export const handleForm = createServerFn({
  method: 'POST',
})
  .validator((data: unknown) => {
    if (!(data instanceof FormData)) {
      throw new Error('Datos de formulario inválidos')
    }
    return data
  })
  .handler(async (ctx) => {
    try {
      const validatedData = await serverValidate(ctx.data)
      console.log('validatedData', validatedData)
      // Persistir los datos del formulario en la base de datos
      // await sql`
      //   INSERT INTO users (name, email, password)
      //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
      // `
    } catch (e) {
      if (e instanceof ServerValidateError) {
        // Registrar errores del formulario o realizar cualquier otra lógica aquí
        return e.response
      }

      // Ocurrió otro error al analizar el formulario
      console.error(e)
      setResponseStatus(500)
      return 'Hubo un error interno'
    }

    return 'El formulario se ha validado correctamente'
  })
```

Luego, necesitamos establecer una forma de obtener los datos del formulario desde la `response` de `serverValidate` usando otra acción de servidor:

```typescript
// app/routes/index.tsx, pero puede extraerse a cualquier otra ruta
import { getFormData } from '@tanstack/react-form/start'

export const getFormDataFromServer = createServerFn({ method: 'GET' }).handler(
  async () => {
    return getFormData()
  },
)
```

Finalmente, usaremos `getFormDataFromServer` en nuestro loader para obtener el estado desde nuestro servidor hacia nuestro cliente y `handleForm` en nuestro componente de formulario del lado del cliente.

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
              ? 'Validación del cliente: Debes tener al menos 8 años'
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
            {isSubmitting ? '...' : 'Enviar'}
          </button>
        )}
      </form.Subscribe>
    </form>
  )
}
```

## Uso de TanStack Form en Next.js con App Router

> Antes de leer esta sección, se sugiere que comprenda cómo funcionan los Componentes de Servidor de React y las Acciones de Servidor de React. [Consulte esta serie de blogs para más información](https://playfulprogramming.com/collections/react-beyond-the-render)

Esta sección se centra en la integración de TanStack Form con `Next.js`, particularmente usando el `App Router` y las `Server Actions`.

### Requisitos previos de Next.js

- Inicie un nuevo proyecto `Next.js`, siguiendo los pasos de la [Documentación de Next.js](https://nextjs.org/docs/getting-started/installation). Asegúrese de seleccionar `sí` para `¿Desea usar App Router?` durante la configuración para acceder a todas las nuevas funciones proporcionadas por Next.js.
- Instale `@tanstack/react-form`
- Instale cualquier [validador de formulario](/form/latest/docs/framework/react/guides/validation#validation-through-schema-libraries) de su elección. [Opcional]

## Integración con App Router

Comencemos creando un `formOption` que usaremos para compartir la estructura del formulario entre el cliente y el servidor.

```typescript
// shared-code.ts
// Observe que la ruta de importación es diferente del cliente
import { formOptions } from '@tanstack/react-form/nextjs'

// Puede pasar otras opciones de formulario aquí
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

A continuación, podemos crear [una Acción de Servidor de React](https://playfulprogramming.com/posts/what-are-react-server-components) que manejará el envío del formulario en el servidor.

```typescript
// action.ts
'use server'

// Observe que la ruta de importación es diferente del cliente
import {
  ServerValidateError,
  createServerValidate,
} from '@tanstack/react-form/nextjs'
import { formOpts } from './shared-code'

// Cree la acción de servidor que inferirá los tipos del formulario desde `formOpts`
const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return 'Validación del servidor: Debes tener al menos 12 años'
    }
  },
})

export default async function someAction(prev: unknown, formData: FormData) {
  try {
    const validatedData = await serverValidate(formData)
    console.log('validatedData', validatedData)
    // Persistir los datos del formulario en la base de datos
    // await sql`
    //   INSERT INTO users (name, email, password)
    //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
    // `
  } catch (e) {
    if (e instanceof ServerValidateError) {
      return e.formState
    }

    // Ocurrió otro error al validar su formulario
    throw e
  }

  // ¡Su formulario se ha validado correctamente!
}
```

Finalmente, usaremos `someAction` en nuestro componente de formulario del lado del cliente.

```tsx
// client-component.tsx
'use client'

import { useActionState } from 'react'
import { initialFormState } from '@tanstack/react-form/nextjs'
// Observe que la importación es desde `react-form`, no `react-form/nextjs`
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
              ? 'Validación del cliente: Debes tener al menos 8 años'
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
            {isSubmitting ? '...' : 'Enviar'}
          </button>
        )}
      </form.Subscribe>
    </form>
  )
}
```

Aquí, estamos usando [el hook `useActionState` de React](https://playfulprogramming.com/posts/what-is-use-action-state-and-form-status) y el hook `useTransform` de TanStack Form para fusionar el estado devuelto desde la acción del servidor con el estado del formulario.

> Si obtiene el siguiente error en su aplicación Next.js:
>
> ```typescript
> x Está importando un componente que necesita `useState`. Este hook de React solo funciona en un componente cliente. Para solucionarlo, marque el archivo (o su padre) con la directiva `"use client"`.
> ```
>
> Esto se debe a que no está importando código del lado del servidor desde `@tanstack/react-form/nextjs`. Asegúrese de importar el módulo correcto según el entorno.
>
> [Esta es una limitación de Next.js](https://github.com/phryneas/rehackt). Otros meta-frameworks probablemente no tendrán este mismo problema.

## Uso de TanStack Form en Remix

> Antes de leer esta sección, se sugiere que comprenda cómo funcionan las acciones de Remix. [Consulte la documentación de Remix para más información](https://remix.run/docs/en/main/discussion/data-flow#route-action)

### Requisitos previos de Remix

- Inicie un nuevo proyecto `Remix`, siguiendo los pasos de la [Documentación de Remix](https://remix.run/docs/en/main/start/quickstart).
- Instale `@tanstack/react-form`
- Instale cualquier [validador de formulario](/form/latest/docs/framework/react/guides/validation#validation-through-schema-libraries) de su elección. [Opcional]

## Integración con Remix

Comencemos creando un `formOption` que usaremos para compartir la estructura del formulario entre el cliente y el servidor.

```typescript
// routes/_index/route.tsx
import { formOptions } from '@tanstack/react-form/remix'

// Puede pasar otras opciones de formulario aquí
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

A continuación, podemos crear [una acción](https://remix.run/docs/en/main/discussion/data-flow#route-action) que manejará el envío del formulario en el servidor.

```tsx
// routes/_index/route.tsx

import {
  ServerValidateError,
  createServerValidate,
  formOptions,
} from '@tanstack/react-form/remix'

import type { ActionFunctionArgs } from '@remix-run/node'

// export const formOpts = formOptions({

// Cree la acción de servidor que inferirá los tipos del formulario desde `formOpts`
const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return 'Validación del servidor: Debes tener al menos 12 años'
    }
  },
})

export async function action({ request }: ActionFunctionArgs) {
  const formData = await request.formData()
  try {
    const validatedData = await serverValidate(formData)
    console.log('validatedData', validatedData)
    // Persistir los datos del formulario en la base de datos
    // await sql`
    //   INSERT INTO users (name, email, password)
    //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
    // `
  } catch (e) {
    if (e instanceof ServerValidateError) {
      return e.formState
    }

    // Ocurrió otro error al validar su formulario
    throw e
  }

  // ¡Su formulario se ha validado correctamente!
}
```

Finalmente, la `action` será llamada cuando se envíe el formulario.

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
              ? 'Validación del cliente: Debes tener al menos 8 años'
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
            {isSubmitting ? '...' : 'Enviar'}
          </button>
        )}
      </form.Subscribe>
    </Form>
  )
}
```

Aquí, estamos usando [el hook `useActionData` de Remix](https://remix.run/docs/en/main/hooks/use-action-data) y el hook `useTransform` de TanStack Form para fusionar el estado devuelto desde la acción del servidor con el estado del formulario.
