---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:38:54.465Z'
id: basic-concepts
title: Concepts de base
---

Cette page présente les concepts de base et la terminologie utilisés dans la bibliothèque `@tanstack/svelte-form`. Se familiariser avec ces concepts vous aidera à mieux comprendre et travailler avec la bibliothèque.

## Options de formulaire (Form Options)

Vous pouvez créer des options pour votre formulaire afin qu'elles puissent être partagées entre plusieurs formulaires en utilisant la fonction `formOptions`.

Exemple :

```ts
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## Instance de formulaire (Form Instance)

Une instance de formulaire (Form Instance) est un objet qui représente un formulaire individuel et fournit des méthodes et des propriétés pour travailler avec le formulaire. Vous créez une instance de formulaire en utilisant la fonction `createForm`. La fonction accepte un objet avec une fonction `onSubmit`, qui est appelée lorsque le formulaire est soumis.

```ts
const form = createForm(() => ({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // Faire quelque chose avec les données du formulaire
    console.log(value)
  },
}))
```

Vous pouvez également créer une instance de formulaire sans utiliser `formOptions` :

```ts
const form = createForm<Person>(() => ({
  onSubmit: async ({ value }) => {
    // Faire quelque chose avec les données du formulaire
    console.log(value)
  },
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  },
}))
```

## Champ (Field)

Un champ (Field) représente un élément de saisie unique dans un formulaire, comme une zone de texte ou une case à cocher. Les champs sont créés en utilisant le composant `form.Field` fourni par l'instance de formulaire. Le composant accepte une prop `name`, qui doit correspondre à une clé dans les valeurs par défaut du formulaire. Il accepte également une prop `children`, qui est une fonction de rendu (render prop) prenant un objet field comme argument.

Exemple :

```svelte
<form.Field name="firstName">
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onblur={field.handleBlur}
      oninput={(e) => field.handleChange(e.target.value)}
    />
  {/snippet}
</form.Field>
```

## État d'un champ (Field State)

Chaque champ possède son propre état (Field State), qui inclut sa valeur actuelle, son statut de validation, les messages d'erreur et d'autres métadonnées. Vous pouvez accéder à l'état d'un champ en utilisant la propriété `field.state`.

Exemple :

```ts
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Il existe trois états de champ très utiles pour comprendre comment l'utilisateur interagit avec un champ. Un champ est _"touched"_ (touché) lorsque l'utilisateur clique ou navigue dedans, _"pristine"_ (vierge) tant que l'utilisateur n'a pas modifié sa valeur, et _"dirty"_ (modifié) après que la valeur a été changée. Vous pouvez vérifier ces états via les indicateurs `isTouched`, `isPristine` et `isDirty`, comme illustré ci-dessous.

```ts
const { isTouched, isPristine, isDirty } = field.state.meta
```

![États d'un champ](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## API d'un champ (Field API)

L'API d'un champ (Field API) est un objet passé à la fonction de rendu lors de la création d'un champ. Il fournit des méthodes pour travailler avec l'état du champ.

Exemple :

```svelte
<input
  name={field.name}
  value={field.state.value}
  onblur={field.handleBlur}
  oninput={(e) => field.handleChange(e.target.value)}
/>
```

## Validation

`@tanstack/svelte-form` fournit à la fois une validation synchrone et asynchrone prête à l'emploi. Les fonctions de validation peuvent être passées au composant `form.Field` en utilisant la prop `validators`.

Exemple :

```svelte
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
>
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onBlur={field.handleBlur}
      onInput={(e) => field.handleChange(e.target.value)}
    />
    <p>{field.state.meta.errors[0]}</p>
  {/snippet}
</form.Field>
```

## Validation avec des bibliothèques de schéma standard

En plus des options de validation personnalisées, nous prenons également en charge la spécification [Standard Schema](https://github.com/standard-schema/standard-schema).

Vous pouvez définir un schéma en utilisant n'importe quelle bibliothèque implémentant cette spécification et le passer à un validateur de formulaire ou de champ.

Les bibliothèques prises en charge incluent :

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```svelte
<script>
  import { z } from 'zod'

  // ...
</script>

<form.Field
  name="firstName"
  validators={{
    onChange: z.string().min(3, 'Le prénom doit comporter au moins 3 caractères'),
    onChangeAsyncDebounceMs: 500,
    onChangeAsync: z.string().refine(
      async (value) => {
        await new Promise((resolve) => setTimeout(resolve, 1000))
        return !value.includes('error')
      },
      {
        message: 'Le prénom ne peut pas contenir "error"',
      },
    ),
  }}
>
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onBlur={field.handleBlur}
      onInput={(e) => field.handleChange(e.target.value)}
    />
    <p>{field.state.meta.errors[0]}</p>
  {/snippet}
</form.Field>
```

## Réactivité (Reactivity)

`@tanstack/svelte-form` offre différentes manières de s'abonner aux changements d'état d'un formulaire ou d'un champ, notamment le hook `form.useStore` et le composant `form.Subscribe`. Ces méthodes vous permettent d'optimiser les performances de rendu de votre formulaire en ne mettant à jour les composants que lorsque nécessaire.

Exemple :

```svelte
<script>
  //...
  const firstName = form.useStore((state) => state.values.firstName)
</script>

<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
>
  {#snippet children(state)}
    <button type="submit" disabled={!state.canSubmit}>
      {state.isSubmitting ? '...' : 'Soumettre'}
    </button>
  {/snippet}
</form.Subscribe>
```

## Champs de tableau (Array Fields)

Les champs de tableau (Array Fields) vous permettent de gérer une liste de valeurs dans un formulaire, comme une liste de hobbies. Vous pouvez créer un champ de tableau en utilisant le composant `form.Field` avec la prop `mode="array"`.

Lorsque vous travaillez avec des champs de tableau, vous pouvez utiliser les méthodes `pushValue`, `removeValue`, `swapValues` et `moveValue` pour ajouter, supprimer et échanger des valeurs dans le tableau.

Exemple :

```svelte
<form.Field name="hobbies" mode="array">
  {#snippet children(hobbiesField)}
    <div>
      Hobbies
      <div>
        {#each hobbiesField.state.value as _, i}
            <div>
              <form.Field name={`hobbies[${i}].name`}>
                {#snippet children(field)}
                  <div>
                    <label for={field.name}>Nom :</label>
                    <input
                      id={field.name}
                      name={field.name}
                      value={field.state.value}
                      onblur={field.handleBlur}
                      onchange={(e) => field.handleChange(e.target.value)}
                    />
                    <button
                      type="button"
                      onclick={() => hobbiesField.removeValue(i)}
                    >
                      X
                    </button>
                  </div>
                {/snippet}
              </form.Field>
              <form.Field name={`hobbies[${i}].description`}>
                {#snippet children(field)}
                    <div>
                      <label for={field.name}>Description :</label>
                      <input
                        id={field.name}
                        name={field.name}
                        value={field.state.value}
                        onblur={field.handleBlur}
                        onchange={(e) => field.handleChange(e.target.value)}
                      />
                    </div>
                {/snippet}
              </form.Field>
            </div>
          {:else}
            Aucun hobby trouvé.
          {/each}
      </div>
      <button
        type="button"
        onclick={() =>
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
  {/snippet}
</form.Field>
```

Voici les concepts de base et la terminologie utilisés dans la bibliothèque `@tanstack/svelte-form`. Comprendre ces concepts vous aidera à travailler plus efficacement avec la bibliothèque et à créer des formulaires complexes avec facilité.
