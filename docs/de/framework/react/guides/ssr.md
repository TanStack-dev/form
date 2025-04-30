---
source-updated-at: '2025-03-28T18:10:56.000Z'
translation-updated-at: '2025-04-30T21:55:41.572Z'
id: ssr
title: SSR/TanStack Start/Next.js
---

TanStack Form ist mit React sofort einsatzbereit, unterstützt `SSR` und ist framework-agnostisch. Allerdings sind spezifische Konfigurationen je nach gewähltem Framework erforderlich.

Aktuell unterstützen wir die folgenden Meta-Frameworks:

- [TanStack Start](https://tanstack.com/start/)
- [Next.js](https://nextjs.org/)
- [Remix](https://remix.run)

## Verwendung von TanStack Form in TanStack Start

Dieser Abschnitt konzentriert sich auf die Integration von TanStack Form mit TanStack Start.

### Voraussetzungen für TanStack Start

- Starten Sie ein neues `TanStack Start`-Projekt, indem Sie die Schritte im [TanStack Start Quickstart Guide](https://tanstack.com/router/latest/docs/framework/react/guide/tanstack-start) befolgen.
- Installieren Sie `@tanstack/react-form`.

### Integration starten

Erstellen wir zunächst eine `formOption`, die wir verwenden, um die Formularstruktur zwischen Client und Server zu teilen.

```typescript
// app/routes/index.tsx, kann aber auch in einem anderen Pfad extrahiert werden
import { formOptions } from '@tanstack/react-form'

// Hier können Sie weitere Formularoptionen übergeben
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

Als Nächstes erstellen wir [eine Start Server Action](https://tanstack.com/start/latest/docs/framework/react/server-functions), die die Formularübermittlung auf dem Server verarbeitet.

```typescript
// app/routes/index.tsx, kann aber auch in einem anderen Pfad extrahiert werden
import {
  createServerValidate,
  ServerValidateError,
} from '@tanstack/react-form/start'

const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return 'Server-Validierung: Sie müssen mindestens 12 Jahre alt sein, um sich anzumelden'
    }
  },
})

export const handleForm = createServerFn({
  method: 'POST',
})
  .validator((data: unknown) => {
    if (!(data instanceof FormData)) {
      throw new Error('Ungültige Formulardaten')
    }
    return data
  })
  .handler(async (ctx) => {
    try {
      const validatedData = await serverValidate(ctx.data)
      console.log('validatedData', validatedData)
      // Formulardaten in der Datenbank speichern
      // await sql`
      //   INSERT INTO users (name, email, password)
      //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
      // `
    } catch (e) {
      if (e instanceof ServerValidateError) {
        // Formularfehler protokollieren oder andere Logik hier ausführen
        return e.response
      }

      // Ein anderer Fehler ist beim Parsen des Formulars aufgetreten
      console.error(e)
      setResponseStatus(500)
      return 'Es ist ein interner Fehler aufgetreten'
    }

    return 'Das Formular wurde erfolgreich validiert'
  })
```

Dann müssen wir eine Möglichkeit schaffen, die Formulardaten aus der `response` von `serverValidate` mithilfe einer weiteren Server-Action abzurufen:

```typescript
// app/routes/index.tsx, kann aber auch in einem anderen Pfad extrahiert werden
import { getFormData } from '@tanstack/react-form/start'

export const getFormDataFromServer = createServerFn({ method: 'GET' }).handler(
  async () => {
    return getFormData()
  },
)
```

Schließlich verwenden wir `getFormDataFromServer` in unserem Loader, um den Zustand vom Server in unseren Client zu übertragen, und `handleForm` in unserer clientseitigen Formularkomponente.

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
              ? 'Client-Validierung: Sie müssen mindestens 8 Jahre alt sein'
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
            {isSubmitting ? '...' : 'Absenden'}
          </button>
        )}
      </form.Subscribe>
    </form>
  )
}
```

## Verwendung von TanStack Form in einem Next.js App Router

> Bevor Sie diesen Abschnitt lesen, wird empfohlen, die Funktionsweise von React Server Components und React Server Actions zu verstehen. [Weitere Informationen finden Sie in dieser Blog-Serie](https://playfulprogramming.com/collections/react-beyond-the-render)

Dieser Abschnitt konzentriert sich auf die Integration von TanStack Form mit `Next.js`, insbesondere unter Verwendung des `App Routers` und `Server Actions`.

### Voraussetzungen für Next.js

- Starten Sie ein neues `Next.js`-Projekt, indem Sie die Schritte in der [Next.js-Dokumentation](https://nextjs.org/docs/getting-started/installation) befolgen. Stellen Sie sicher, dass Sie während der Einrichtung `Ja` für `Would you like to use App Router?` auswählen, um auf alle neuen Funktionen von Next.js zugreifen zu können.
- Installieren Sie `@tanstack/react-form`
- Installieren Sie optional eine [Formularvalidierungsbibliothek](/form/latest/docs/framework/react/guides/validation#validation-through-schema-libraries) Ihrer Wahl.

## Integration des App Routers

Erstellen wir zunächst eine `formOption`, die wir verwenden, um die Formularstruktur zwischen Client und Server zu teilen.

```typescript
// shared-code.ts
// Beachten Sie, dass der Importpfad vom Client abweicht
import { formOptions } from '@tanstack/react-form/nextjs'

// Hier können Sie weitere Formularoptionen übergeben
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

Als Nächstes erstellen wir [eine React Server Action](https://playfulprogramming.com/posts/what-are-react-server-components), die die Formularübermittlung auf dem Server verarbeitet.

```typescript
// action.ts
'use server'

// Beachten Sie, dass der Importpfad vom Client abweicht
import {
  ServerValidateError,
  createServerValidate,
} from '@tanstack/react-form/nextjs'
import { formOpts } from './shared-code'

// Erstellen Sie die Server-Action, die die Typen des Formulars aus `formOpts` ableitet
const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return 'Server-Validierung: Sie müssen mindestens 12 Jahre alt sein'
    }
  },
})

export default async function someAction(prev: unknown, formData: FormData) {
  try {
    const validatedData = await serverValidate(formData)
    console.log('validatedData', validatedData)
    // Formulardaten in der Datenbank speichern
    // await sql`
    //   INSERT INTO users (name, email, password)
    //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
    // `
  } catch (e) {
    if (e instanceof ServerValidateError) {
      return e.formState
    }

    // Ein anderer Fehler ist beim Validieren des Formulars aufgetreten
    throw e
  }

  // Ihr Formular wurde erfolgreich validiert!
}
```

Schließlich verwenden wir `someAction` in unserer clientseitigen Formularkomponente.

```tsx
// client-component.tsx
'use client'

import { useActionState } from 'react'
import { initialFormState } from '@tanstack/react-form/nextjs'
// Beachten Sie, dass der Import von `react-form` stammt, nicht von `react-form/nextjs`
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
              ? 'Client-Validierung: Sie müssen mindestens 8 Jahre alt sein'
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
            {isSubmitting ? '...' : 'Absenden'}
          </button>
        )}
      </form.Subscribe>
    </form>
  )
}
```

Hier verwenden wir [React's `useActionState` Hook](https://playfulprogramming.com/posts/what-is-use-action-state-and-form-status) und TanStack Form's `useTransform` Hook, um den vom Server zurückgegebenen Zustand mit dem Formularzustand zu mergen.

> Wenn Sie in Ihrer Next.js-Anwendung den folgenden Fehler erhalten:
>
> ```typescript
> x Sie importieren eine Komponente, die `useState` benötigt. Dieser React Hook funktioniert nur in einer Client-Komponente. Fügen Sie zur Behebung die Direktive `"use client"` in der Datei (oder ihrer übergeordneten Datei) hinzu.
> ```
>
> Dies liegt daran, dass Sie serverseitigen Code nicht von `@tanstack/react-form/nextjs` importieren. Stellen Sie sicher, dass Sie das richtige Modul basierend auf der Umgebung importieren.
>
> [Dies ist eine Einschränkung von Next.js](https://github.com/phryneas/rehackt). Andere Meta-Frameworks haben dieses Problem wahrscheinlich nicht.

## Verwendung von TanStack Form in Remix

> Bevor Sie diesen Abschnitt lesen, wird empfohlen, die Funktionsweise von Remix-Actions zu verstehen. [Weitere Informationen finden Sie in der Remix-Dokumentation](https://remix.run/docs/en/main/discussion/data-flow#route-action)

### Voraussetzungen für Remix

- Starten Sie ein neues `Remix`-Projekt, indem Sie die Schritte in der [Remix-Dokumentation](https://remix.run/docs/en/main/start/quickstart) befolgen.
- Installieren Sie `@tanstack/react-form`
- Installieren Sie optional eine [Formularvalidierungsbibliothek](/form/latest/docs/framework/react/guides/validation#validation-through-schema-libraries) Ihrer Wahl.

## Integration in Remix

Erstellen wir zunächst eine `formOption`, die wir verwenden, um die Formularstruktur zwischen Client und Server zu teilen.

```typescript
// routes/_index/route.tsx
import { formOptions } from '@tanstack/react-form/remix'

// Hier können Sie weitere Formularoptionen übergeben
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

Als Nächstes erstellen wir [eine Action](https://remix.run/docs/en/main/discussion/data-flow#route-action), die die Formularübermittlung auf dem Server verarbeitet.

```tsx
// routes/_index/route.tsx

import {
  ServerValidateError,
  createServerValidate,
  formOptions,
} from '@tanstack/react-form/remix'

import type { ActionFunctionArgs } from '@remix-run/node'

// export const formOpts = formOptions({

// Erstellen Sie die Server-Action, die die Typen des Formulars aus `formOpts` ableitet
const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return 'Server-Validierung: Sie müssen mindestens 12 Jahre alt sein'
    }
  },
})

export async function action({ request }: ActionFunctionArgs) {
  const formData = await request.formData()
  try {
    const validatedData = await serverValidate(formData)
    console.log('validatedData', validatedData)
    // Formulardaten in der Datenbank speichern
    // await sql`
    //   INSERT INTO users (name, email, password)
    //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
    // `
  } catch (e) {
    if (e instanceof ServerValidateError) {
      return e.formState
    }

    // Ein anderer Fehler ist beim Validieren des Formulars aufgetreten
    throw e
  }

  // Ihr Formular wurde erfolgreich validiert!
}
```

Schließlich wird die `action` beim Absenden des Formulars aufgerufen.

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
              ? 'Client-Validierung: Sie müssen mindestens 8 Jahre alt sein'
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
            {isSubmitting ? '...' : 'Absenden'}
          </button>
        )}
      </form.Subscribe>
    </Form>
  )
}
```

Hier verwenden wir [Remix's `useActionData` Hook](https://remix.run/docs/en/main/hooks/use-action-data) und TanStack Form's `useTransform` Hook, um den vom Server zurückgegebenen Zustand mit dem Formularzustand zu mergen.
