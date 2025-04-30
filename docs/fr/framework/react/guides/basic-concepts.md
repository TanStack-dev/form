---
source-updated-at: '2025-04-16T08:45:06.000Z'
translation-updated-at: '2025-04-30T21:36:23.772Z'
id: basic-concepts
title: Concepts de base
---

Cette page présente les concepts de base et la terminologie utilisés dans la bibliothèque `@tanstack/react-form`. Se familiariser avec ces concepts vous aidera à mieux comprendre et travailler avec la bibliothèque.

## Options de formulaire

Vous pouvez créer des options pour votre formulaire afin qu'elles puissent être partagées entre plusieurs formulaires en utilisant la fonction `formOptions`.

Exemple :

```tsx
interface User {
  firstName: string
  lastName: string
  hobbies: Array<string>
}
const defaultUser: User = { firstName: '', lastName: '', hobbies: [] }

const formOpts = formOptions({
  defaultValues: defaultUser,
})
```

## Instance de formulaire

Une Instance de formulaire est un objet qui représente un formulaire individuel et fournit des méthodes et propriétés pour travailler avec le formulaire. Vous créez une instance de formulaire en utilisant le hook `useForm` fourni par les options de formulaire. Le hook accepte un objet avec une fonction `onSubmit`, qui est appelée lorsque le formulaire est soumis.

```tsx
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // Faire quelque chose avec les données du formulaire
    console.log(value)
  },
})
```

Vous pouvez également créer une instance de formulaire sans utiliser `formOptions` en utilisant l'API autonome `useForm` :

```tsx
interface User {
  firstName: string
  lastName: string
  hobbies: Array<string>
}
const defaultUser: User = { firstName: '', lastName: '', hobbies: [] }

const form = useForm({
  defaultValues: defaultUser,
  onSubmit: async ({ value }) => {
    // Faire quelque chose avec les données du formulaire
    console.log(value)
  },
})
```

## Champ

Un Champ représente un seul élément de saisie de formulaire, comme un champ de texte ou une case à cocher. Les champs sont créés en utilisant le composant `form.Field` fourni par l'instance de formulaire. Le composant accepte une prop `name`, qui doit correspondre à une clé dans les valeurs par défaut du formulaire. Il accepte également une prop `children`, qui est une fonction de rendu qui prend un objet champ comme argument.

Exemple :

```tsx
<form.Field
  name="firstName"
  children={(field) => (
    <>
      <input
        value={field.state.value}
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.value)}
      />
      <FieldInfo field={field} />
    </>
  )}
/>
```

## État d'un champ

Chaque champ a son propre état, qui inclut sa valeur actuelle, son statut de validation, ses messages d'erreur et d'autres métadonnées. Vous pouvez accéder à l'état d'un champ en utilisant la propriété `field.state`.

Exemple :

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Il existe trois états de champ qui peuvent être utiles pour voir comment l'utilisateur interagit avec un champ : un champ est _"touché"_ lorsque l'utilisateur clique/tabule dessus, _"vierge"_ jusqu'à ce que l'utilisateur modifie sa valeur, et _"modifié"_ après que la valeur a été changée. Vous pouvez vérifier ces états via les indicateurs `isTouched`, `isPristine` et `isDirty`, comme vu ci-dessous.

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![États d'un champ](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

> **Note importante pour les utilisateurs venant de `React Hook Form`** : l'indicateur `isDirty` dans `TanStack/form` est différent de l'indicateur du même nom dans RHF.
> Dans RHF, `isDirty = true` lorsque les valeurs du formulaire sont différentes des valeurs originales. Si l'utilisateur modifie les valeurs d'un formulaire, puis les modifie à nouveau pour revenir aux valeurs par défaut du formulaire, `isDirty` sera `false` dans RHF, mais `true` dans `TanStack/form`.
> Les valeurs par défaut sont exposées à la fois au niveau du formulaire et du champ dans `TanStack/form` (`form.options.defaultValues`, `field.options.defaultValue`), vous pouvez donc écrire votre propre helper `isDefaultValue()` si vous avez besoin d'émuler le comportement de RHF.

## API de champ

L'API de champ est un objet passé à la fonction de rendu lors de la création d'un champ. Elle fournit des méthodes pour travailler avec l'état du champ.

Exemple :

```tsx
<input
  value={field.state.value}
  onBlur={field.handleBlur}
  onChange={(e) => field.handleChange(e.target.value)}
/>
```

## Validation

`@tanstack/react-form` fournit à la fois une validation synchrone et asynchrone prête à l'emploi. Les fonctions de validation peuvent être passées au composant `form.Field` en utilisant la prop `validators`.

Exemple :

```tsx
<form.Field
  name="firstName"
  validators={{
    onChange: ({ value }) =>
      !value
        ? 'Un prénom est requis'
        : value.length < 3
          ? 'Le prénom doit comporter au moins 3 caractères'
          : undefined,
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && 'Le prénom ne peut pas contenir "error"'
    },
  }}
  children={(field) => (
    <>
      <input
        value={field.state.value}
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.value)}
      />
      <FieldInfo field={field} />
    </>
  )}
/>
```

## Validation avec des bibliothèques de schéma standard

En plus des options de validation personnalisées, nous prenons également en charge la spécification [Standard Schema](https://github.com/standard-schema/standard-schema).

Vous pouvez définir un schéma en utilisant n'importe quelle bibliothèque implémentant la spécification et le passer à un validateur de formulaire ou de champ.

Les bibliothèques prises en charge incluent :

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```tsx
import { z } from 'zod'

const userSchema = z.object({
  age: z.number().gte(13, 'Vous devez avoir 13 ans pour créer un compte'),
})

function App() {
  const form = useForm({
    defaultValues: {
      age: 0,
    },
    validators: {
      onChange: userSchema,
    },
  })
  return (
    <div>
      <form.Field
        name="age"
        children={(field) => {
          return <>{/* ... */}</>
        }}
      />
    </div>
  )
}
```

## Réactivité

`@tanstack/react-form` offre diverses façons de s'abonner aux changements d'état du formulaire et des champs, notamment le hook `useStore(form.store)` et le composant `form.Subscribe`. Ces méthodes vous permettent d'optimiser les performances de rendu de votre formulaire en ne mettant à jour les composants que lorsque nécessaire.

Exemple :

```tsx
const firstName = useStore(form.store, (state) => state.values.firstName)
//...
<form.Subscribe
  selector={(state) => [state.canSubmit, state.isSubmitting]}
  children={([canSubmit, isSubmitting]) => (
    <button type="submit" disabled={!canSubmit}>
      {isSubmitting ? '...' : 'Soumettre'}
    </button>
  )}
/>
```

Il est important de se rappeler que bien que la prop `selector` du hook `useStore` soit optionnelle, il est fortement recommandé de la fournir, car son omission entraînera des re-rendus inutiles.

```tsx
// Utilisation correcte
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
// Utilisation incorrecte
const store = useStore(form.store)
```

Remarque : L'utilisation du hook `useField` pour obtenir de la réactivité est déconseillée car il est conçu pour être utilisé avec précaution dans le composant `form.Field`. Vous devriez plutôt utiliser `useStore(form.store)`.

## Écouteurs

`@tanstack/react-form` vous permet de réagir à des déclencheurs spécifiques et de les "écouter" pour déclencher des effets secondaires.

Exemple :

```tsx
<form.Field
  name="country"
  listeners={{
    onChange: ({ value }) => {
      console.log(`Pays changé en : ${value}, réinitialisation de la province`)
      form.setFieldValue('province', '')
    },
  }}
/>
```

Plus d'informations peuvent être trouvées sur [Écouteurs](./listeners.md)

## Champs de tableau

Les champs de tableau vous permettent de gérer une liste de valeurs dans un formulaire, comme une liste de hobbies. Vous pouvez créer un champ de tableau en utilisant le composant `form.Field` avec la prop `mode="array"`.

Lorsque vous travaillez avec des champs de tableau, vous pouvez utiliser les méthodes `pushValue`, `removeValue`, `swapValues` et `moveValue` des champs pour ajouter, supprimer et échanger des valeurs dans le tableau.

Exemple :

```tsx
<form.Field
  name="hobbies"
  mode="array"
  children={(hobbiesField) => (
    <div>
      Hobbies
      <div>
        {!hobbiesField.state.value.length
          ? 'Aucun hobby trouvé.'
          : hobbiesField.state.value.map((_, i) => (
              <div key={i}>
                <form.Field
                  name={`hobbies[${i}].name`}
                  children={(field) => {
                    return (
                      <div>
                        <label htmlFor={field.name}>Nom :</label>
                        <input
                          id={field.name}
                          name={field.name}
                          value={field.state.value}
                          onBlur={field.handleBlur}
                          onChange={(e) => field.handleChange(e.target.value)}
                        />
                        <button
                          type="button"
                          onClick={() => hobbiesField.removeValue(i)}
                        >
                          X
                        </button>
                        <FieldInfo field={field} />
                      </div>
                    )
                  }}
                />
                <form.Field
                  name={`hobbies[${i}].description`}
                  children={(field) => {
                    return (
                      <div>
                        <label htmlFor={field.name}>Description :</label>
                        <input
                          id={field.name}
                          name={field.name}
                          value={field.state.value}
                          onBlur={field.handleBlur}
                          onChange={(e) => field.handleChange(e.target.value)}
                        />
                        <FieldInfo field={field} />
                      </div>
                    )
                  }}
                />
              </div>
            ))}
      </div>
      <button
        type="button"
        onClick={() =>
          hobbiesField.pushValue({
            name: '',
            description: '',
            yearsOfExperience: 0,
          })
        }
      >
        Ajouter un hobby
      </button>
    </div>
  )}
/>
```

## Boutons de réinitialisation

Lorsque vous utilisez `<button type="reset">` en conjonction avec `form.reset()` de TanStack Form, vous devez empêcher le comportement de réinitialisation HTML par défaut pour éviter des réinitialisations inattendues des éléments de formulaire (en particulier les éléments `<select>`) à leurs valeurs HTML initiales.
Utilisez `event.preventDefault()` dans le gestionnaire `onClick` du bouton pour empêcher la réinitialisation native du formulaire.

Exemple :

```tsx
<button
  type="reset"
  onClick={(event) => {
    event.preventDefault()
    form.reset()
  }}
>
  Réinitialiser
</button>
```

Alternativement, vous pouvez utiliser `<button type="button">` pour empêcher la réinitialisation HTML native.

```tsx
<button
  type="button"
  onClick={() => {
    form.reset()
  }}
>
  Réinitialiser
</button>
```

Voici les concepts de base et la terminologie utilisés dans la bibliothèque `@tanstack/react-form`. Comprendre ces concepts vous aidera à travailler plus efficacement avec la bibliothèque et à créer des formulaires complexes avec facilité.
