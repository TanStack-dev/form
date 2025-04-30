---
source-updated-at: '2025-03-28T18:10:56.000Z'
translation-updated-at: '2025-04-30T21:37:41.939Z'
id: ssr
title: SSR/TanStack Start/Next.js
---

# Utilisation des Méta-Frameworks React avec TanStack Form

TanStack Form est compatible avec React dès le départ, prenant en charge le `SSR` (Rendu côté serveur) et étant indépendant du framework. Cependant, des configurations spécifiques sont nécessaires selon le framework choisi.

Nous prenons actuellement en charge les méta-frameworks suivants :

- [TanStack Start](https://tanstack.com/start/)
- [Next.js](https://nextjs.org/)
- [Remix](https://remix.run)

## Utilisation de TanStack Form avec TanStack Start

Cette section se concentre sur l'intégration de TanStack Form avec TanStack Start.

### Prérequis pour TanStack Start

- Créez un nouveau projet `TanStack Start` en suivant les étapes du [Guide de démarrage rapide de TanStack Start](https://tanstack.com/router/latest/docs/framework/react/guide/tanstack-start)
- Installez `@tanstack/react-form`

### Intégration

Commençons par créer une `formOption` que nous utiliserons pour partager la structure du formulaire entre le client et le serveur.

```typescript
// app/routes/index.tsx, mais peut être extrait vers n'importe quel autre chemin
import { formOptions } from '@tanstack/react-form'

// Vous pouvez passer d'autres options de formulaire ici
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

Ensuite, nous pouvons créer [une Action Serveur Start](https://tanstack.com/start/latest/docs/framework/react/server-functions) qui gérera la soumission du formulaire côté serveur.

```typescript
// app/routes/index.tsx, mais peut être extrait vers n'importe quel autre chemin
import {
  createServerValidate,
  ServerValidateError,
} from '@tanstack/react-form/start'

const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return 'Validation serveur : Vous devez avoir au moins 12 ans pour vous inscrire'
    }
  },
})

export const handleForm = createServerFn({
  method: 'POST',
})
  .validator((data: unknown) => {
    if (!(data instanceof FormData)) {
      throw new Error('Données de formulaire invalides')
    }
    return data
  })
  .handler(async (ctx) => {
    try {
      const validatedData = await serverValidate(ctx.data)
      console.log('validatedData', validatedData)
      // Persistez les données du formulaire dans la base de données
      // await sql`
      //   INSERT INTO users (name, email, password)
      //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
      // `
    } catch (e) {
      if (e instanceof ServerValidateError) {
        // Enregistrez les erreurs du formulaire ou effectuez toute autre logique ici
        return e.response
      }

      // Une autre erreur s'est produite lors de l'analyse du formulaire
      console.error(e)
      setResponseStatus(500)
      return 'Une erreur interne est survenue'
    }

    return 'Le formulaire a été validé avec succès'
  })
```

Ensuite, nous devons établir un moyen de récupérer les données du formulaire à partir de la `response` de `serverValidate` en utilisant une autre action serveur :

```typescript
// app/routes/index.tsx, mais peut être extrait vers n'importe quel autre chemin
import { getFormData } from '@tanstack/react-form/start'

export const getFormDataFromServer = createServerFn({ method: 'GET' }).handler(
  async () => {
    return getFormData()
  },
)
```

Enfin, nous utiliserons `getFormDataFromServer` dans notre loader pour obtenir l'état de notre serveur vers notre client et `handleForm` dans notre composant de formulaire côté client.

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
              ? 'Validation client : Vous devez avoir au moins 8 ans'
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
            {isSubmitting ? '...' : 'Soumettre'}
          </button>
        )}
      </form.Subscribe>
    </form>
  )
}
```

## Utilisation de TanStack Form avec Next.js App Router

> Avant de lire cette section, il est recommandé de comprendre comment fonctionnent les Composants Serveur React et les Actions Serveur React. [Consultez cette série de blogs pour plus d'informations](https://playfulprogramming.com/collections/react-beyond-the-render)

Cette section se concentre sur l'intégration de TanStack Form avec `Next.js`, en particulier en utilisant l'`App Router` et les `Server Actions`.

### Prérequis pour Next.js

- Créez un nouveau projet `Next.js` en suivant les étapes de la [Documentation Next.js](https://nextjs.org/docs/getting-started/installation). Assurez-vous de sélectionner `oui` pour `Would you like to use App Router?` pendant la configuration pour accéder à toutes les nouvelles fonctionnalités fournies par Next.js.
- Installez `@tanstack/react-form`
- Installez n'importe quel [validateur de formulaire](/form/latest/docs/framework/react/guides/validation#validation-through-schema-libraries) de votre choix. [Optionnel]

## Intégration avec App Router

Commençons par créer une `formOption` que nous utiliserons pour partager la structure du formulaire entre le client et le serveur.

```typescript
// shared-code.ts
// Notez que le chemin d'importation est différent du client
import { formOptions } from '@tanstack/react-form/nextjs'

// Vous pouvez passer d'autres options de formulaire ici
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

Ensuite, nous pouvons créer [une Action Serveur React](https://playfulprogramming.com/posts/what-are-react-server-components) qui gérera la soumission du formulaire côté serveur.

```typescript
// action.ts
'use server'

// Notez que le chemin d'importation est différent du client
import {
  ServerValidateError,
  createServerValidate,
} from '@tanstack/react-form/nextjs'
import { formOpts } from './shared-code'

// Créez l'action serveur qui déduira les types du formulaire à partir de `formOpts`
const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return 'Validation serveur : Vous devez avoir au moins 12 ans'
    }
  },
})

export default async function someAction(prev: unknown, formData: FormData) {
  try {
    const validatedData = await serverValidate(formData)
    console.log('validatedData', validatedData)
    // Persistez les données du formulaire dans la base de données
    // await sql`
    //   INSERT INTO users (name, email, password)
    //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
    // `
  } catch (e) {
    if (e instanceof ServerValidateError) {
      return e.formState
    }

    // Une autre erreur s'est produite lors de la validation de votre formulaire
    throw e
  }

  // Votre formulaire a été validé avec succès !
}
```

Enfin, nous utiliserons `someAction` dans notre composant de formulaire côté client.

```tsx
// client-component.tsx
'use client'

import { useActionState } from 'react'
import { initialFormState } from '@tanstack/react-form/nextjs'
// Notez que l'import provient de `react-form`, pas de `react-form/nextjs`
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
              ? 'Validation client : Vous devez avoir au moins 8 ans'
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
            {isSubmitting ? '...' : 'Soumettre'}
          </button>
        )}
      </form.Subscribe>
    </form>
  )
}
```

Ici, nous utilisons [le hook `useActionState` de React](https://playfulprogramming.com/posts/what-is-use-action-state-and-form-status) et le hook `useTransform` de TanStack Form pour fusionner l'état renvoyé par l'action serveur avec l'état du formulaire.

> Si vous obtenez l'erreur suivante dans votre application Next.js :
>
> ```typescript
> x Vous importez un composant qui nécessite `useState`. Ce hook React ne fonctionne que dans un composant client. Pour corriger, marquez le fichier (ou son parent) avec la directive `"use client"`.
> ```
>
> C'est parce que vous n'importez pas le code côté serveur depuis `@tanstack/react-form/nextjs`. Assurez-vous d'importer le module correct en fonction de l'environnement.
>
> [Il s'agit d'une limitation de Next.js](https://github.com/phryneas/rehackt). D'autres méta-frameworks n'auront probablement pas ce même problème.

## Utilisation de TanStack Form avec Remix

> Avant de lire cette section, il est recommandé de comprendre comment fonctionnent les actions Remix. [Consultez la documentation de Remix pour plus d'informations](https://remix.run/docs/en/main/discussion/data-flow#route-action)

### Prérequis pour Remix

- Créez un nouveau projet `Remix` en suivant les étapes de la [Documentation Remix](https://remix.run/docs/en/main/start/quickstart).
- Installez `@tanstack/react-form`
- Installez n'importe quel [validateur de formulaire](/form/latest/docs/framework/react/guides/validation#validation-through-schema-libraries) de votre choix. [Optionnel]

## Intégration avec Remix

Commençons par créer une `formOption` que nous utiliserons pour partager la structure du formulaire entre le client et le serveur.

```typescript
// routes/_index/route.tsx
import { formOptions } from '@tanstack/react-form/remix'

// Vous pouvez passer d'autres options de formulaire ici
export const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    age: 0,
  },
})
```

Ensuite, nous pouvons créer [une action](https://remix.run/docs/en/main/discussion/data-flow#route-action) qui gérera la soumission du formulaire côté serveur.

```tsx
// routes/_index/route.tsx

import {
  ServerValidateError,
  createServerValidate,
  formOptions,
} from '@tanstack/react-form/remix'

import type { ActionFunctionArgs } from '@remix-run/node'

// export const formOpts = formOptions({

// Créez l'action serveur qui déduira les types du formulaire à partir de `formOpts`
const serverValidate = createServerValidate({
  ...formOpts,
  onServerValidate: ({ value }) => {
    if (value.age < 12) {
      return 'Validation serveur : Vous devez avoir au moins 12 ans'
    }
  },
})

export async function action({ request }: ActionFunctionArgs) {
  const formData = await request.formData()
  try {
    const validatedData = await serverValidate(formData)
    console.log('validatedData', validatedData)
    // Persistez les données du formulaire dans la base de données
    // await sql`
    //   INSERT INTO users (name, email, password)
    //   VALUES (${validatedData.name}, ${validatedData.email}, ${validatedData.password})
    // `
  } catch (e) {
    if (e instanceof ServerValidateError) {
      return e.formState
    }

    // Une autre erreur s'est produite lors de la validation de votre formulaire
    throw e
  }

  // Votre formulaire a été validé avec succès !
}
```

Enfin, l'`action` sera appelée lorsque le formulaire est soumis.

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
              ? 'Validation client : Vous devez avoir au moins 8 ans'
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
            {isSubmitting ? '...' : 'Soumettre'}
          </button>
        )}
      </form.Subscribe>
    </Form>
  )
}
```

Ici, nous utilisons [le hook `useActionData` de Remix](https://remix.run/docs/en/main/hooks/use-action-data) et le hook `useTransform` de TanStack Form pour fusionner l'état renvoyé par l'action serveur avec l'état du formulaire.
