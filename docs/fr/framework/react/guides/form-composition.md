---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:37:04.659Z'
id: form-composition
title: Composition de formulaire
---

# Composition de formulaires

Une critique courante de TanStack Form est sa verbosité par défaut. Bien que cela _puisse_ être utile à des fins éducatives - en aidant à renforcer la compréhension de nos APIs - ce n'est pas idéal pour les cas d'utilisation en production.

Par conséquent, bien que `form.Field` permette l'utilisation la plus puissante et flexible de TanStack Form, nous fournissons des APIs qui l'encapsulent et rendent votre code d'application moins verbeux.

## Hooks de formulaire personnalisés

La manière la plus puissante de composer des formulaires est de créer des hooks de formulaire personnalisés. Cela vous permet de créer un hook de formulaire adapté aux besoins de votre application, incluant des composants d'interface utilisateur pré-liés et plus encore.

À sa forme la plus basique, `createFormHook` est une fonction qui prend un `fieldContext` et un `formContext` et retourne un hook `useAppForm`.

> Ce hook `useAppForm` non personnalisé est identique à `useForm`, mais cela changera rapidement lorsque nous ajouterons plus d'options à `createFormHook`.

```tsx
import { createFormHookContexts, createFormHook } from '@tanstack/react-form'

// export useFieldContext pour utilisation dans vos composants personnalisés
export const { fieldContext, formContext, useFieldContext } =
  createFormHookContexts()

const { useAppForm } = createFormHook({
  fieldContext,
  formContext,
  // Nous en apprendrons plus sur ces options plus tard
  fieldComponents: {},
  formComponents: {},
})

function App() {
  const form = useAppForm({
    // Prend en charge toutes les options de useForm
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return <form.Field /> // ...
}
```

### Composants de champ pré-liés

Une fois cette structure en place, vous pouvez commencer à ajouter des composants de champ et de formulaire personnalisés à votre hook de formulaire.

> Note : le `useFieldContext` doit être le même que celui exporté depuis votre contexte de formulaire personnalisé

```tsx
import { useFieldContext } from './form-context.tsx'

export function TextField({ label }: { label: string }) {
  // Le `Field` déduit qu'il devrait avoir un type `value` de `string`
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

Vous pouvez ensuite enregistrer ce composant avec votre hook de formulaire.

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

Et l'utiliser dans votre formulaire :

```tsx
function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return (
    // Remarquez le `AppField` au lieu de `Field` ; `AppField` fournit le contexte requis
    <form.AppField
      name="firstName"
      children={(field) => <field.TextField label="First Name" />}
    />
  )
}
```

Cela vous permet non seulement de réutiliser l'interface utilisateur de votre composant partagé, mais conserve également la sécurité de type que vous attendez de TanStack Form : une faute de frappe dans `name` générera une erreur TypeScript.

#### Une note sur les performances

Bien que le contexte soit un outil précieux dans l'écosystème React, de nombreux utilisateurs expriment à juste titre des inquiétudes quant au fait que fournir une valeur réactive via un contexte peut causer des rendus inutiles.

> Pas familier avec cette préoccupation de performance ? [L'article de blog de Mark Erikson expliquant pourquoi Redux résout beaucoup de ces problèmes](https://blog.isquaredsoftware.com/2021/01/context-redux-differences/) est un excellent point de départ.

Bien que ce soit une préoccupation légitime à souligner, ce n'est pas un problème pour TanStack Form ; les valeurs fournies via le contexte ne sont pas réactives elles-mêmes, mais sont plutôt des instances de classe statiques avec des propriétés réactives ([utilisant TanStack Store comme implémentation de signaux pour alimenter le tout](https://tanstack.com/store)).

### Composants de formulaire pré-liés

Bien que `form.AppField` résolve de nombreux problèmes liés au boilerplate des champs et à la réutilisabilité, il ne résout pas le problème du boilerplate des _formulaires_ et de leur réutilisabilité.

En particulier, pouvoir partager des instances de `form.Subscribe` pour, par exemple, un bouton de soumission réactif est un cas d'utilisation courant.

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
      // Remarquez le composant wrapper `AppForm` ; `AppForm` fournit le
      contexte requis
      <form.SubscribeButton label="Submit" />
    </form.AppForm>
  )
}
```

## Décomposer les grands formulaires en plus petites parties

Parfois, les formulaires deviennent très grands ; c'est comme ça parfois. Bien que TanStack Form prenne bien en charge les grands formulaires, ce n'est jamais agréable de travailler avec des fichiers de centaines ou de milliers de lignes de code.

Pour résoudre ce problème, nous prenons en charge la décomposition des formulaires en plus petites parties en utilisant le composant d'ordre supérieur `withForm`.

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
  // Ces valeurs ne sont utilisées que pour la vérification de type, et ne sont pas utilisées à l'exécution
  // Cela vous permet de `...formOpts` depuis `formOptions` sans avoir à redéclarer les options
  defaultValues: {
    firstName: 'John',
    lastName: 'Doe',
  },
  // Optionnel, mais ajoute des props à la fonction `render` en plus de `form`
  props: {
    // Ces props sont également définies comme valeurs par défaut pour la fonction `render`
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

### FAQ sur `withForm`

> Pourquoi un composant d'ordre supérieur au lieu d'un hook ?

Bien que les hooks soient l'avenir de React, les composants d'ordre supérieur restent un outil puissant pour la composition. En particulier, l'API de `useForm` nous permet d'avoir une forte sécurité de type sans obliger les utilisateurs à passer des génériques.

> Pourquoi est-ce que je reçois des erreurs ESLint concernant les hooks dans `render` ?

ESLint recherche les hooks au niveau supérieur d'une fonction, et `render` peut ne pas être reconnu comme un composant de niveau supérieur, selon la façon dont vous l'avez défini.

```tsx
// Cela causera des erreurs ESLint avec l'utilisation de hooks
const ChildForm = withForm({
  // ...
  render: ({ form, title }) => {
    // ...
  },
})
```

```tsx
// Cela fonctionne bien
const ChildForm = withForm({
  // ...
  render: function Render({ form, title }) {
    // ...
  },
})
```

## Tree-shaking des composants de formulaire et de champ

Bien que les exemples ci-dessus soient excellents pour commencer, ils ne sont pas idéaux pour certains cas d'utilisation où vous pourriez avoir des centaines de composants de formulaire et de champ.
En particulier, vous ne voudrez peut-être pas inclure tous vos composants de formulaire et de champ dans le bundle de chaque fichier utilisant votre hook de formulaire.

Pour résoudre ce problème, vous pouvez combiner l'API TanStack `createFormHook` avec les composants React `lazy` et `Suspense` :

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

Cela affichera le fallback Suspense pendant que le composant `TextField` est en cours de chargement, puis rendra le formulaire une fois qu'il est chargé.

## Tout mettre ensemble

Maintenant que nous avons couvert les bases de la création de hooks de formulaire personnalisés, mettons tout cela ensemble dans un seul exemple.

```tsx
// /src/hooks/form.ts, à utiliser dans toute l'application
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

// /src/features/people/shared-form.ts, à utiliser dans les fonctionnalités `people`
const formOpts = formOptions({
  defaultValues: {
    firstName: 'John',
    lastName: 'Doe',
  },
})

// /src/features/people/nested-form.ts, à utiliser dans la page `people`
const ChildForm = withForm({
  ...formOpts,
  // Optionnel, mais ajoute des props à la fonction `render` en dehors de `form`
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

## Guide d'utilisation de l'API

Voici un tableau pour vous aider à décider quelles APIs vous devriez utiliser :

![](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/react_form_composability.svg)
