---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:37:08.881Z'
id: basic-concepts
title: Concepts de base
---

Cette page présente les concepts de base et la terminologie utilisés dans la bibliothèque `@tanstack/vue-form`. Se familiariser avec ces concepts vous aidera à mieux comprendre et travailler avec la bibliothèque.

## Options de formulaire

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

## Instance de formulaire

Une Instance de formulaire est un objet qui représente un formulaire individuel et fournit des méthodes et propriétés pour travailler avec le formulaire. Vous créez une instance de formulaire en utilisant la fonction `useForm`. La fonction accepte un objet avec une fonction `onSubmit`, qui est appelée lorsque le formulaire est soumis.

```js
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // Faire quelque chose avec les données du formulaire
    console.log(value)
  },
})
```

Vous pouvez également créer une instance de formulaire sans utiliser `formOptions` en utilisant l'API autonome `useForm` :

```ts
const form = useForm({
  onSubmit: async ({ value }) => {
    // Faire quelque chose avec les données du formulaire
    console.log(value)
  },
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## Champ

Un Champ représente un seul élément de saisie de formulaire, comme une entrée de texte ou une case à cocher. Les champs sont créés en utilisant le composant `form.Field` fourni par l'instance de formulaire. Le composant accepte une prop `name`, qui doit correspondre à une clé dans les valeurs par défaut du formulaire. Il accepte également un slot scopé défini par la directive `v-slot` qui prend un objet `field` comme argument.

Exemple :

```vue
<template>
  <!-- ... -->
  <form.Field name="fullName">
    <template v-slot="{ field }">
      <input
        :name="field.name"
        :value="field.state.value"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange(e.target.value)"
      />
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## État d'un champ

Chaque champ a son propre état, qui inclut sa valeur actuelle, son statut de validation, les messages d'erreur et d'autres métadonnées. Vous pouvez accéder à l'état d'un champ en utilisant la propriété `field.state`.

Exemple :

```js
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Il existe trois états de champ qui peuvent être très utiles pour voir comment l'utilisateur interagit avec un champ. Un champ est _"touched"_ (touché) lorsque l'utilisateur clique ou tabule dessus, _"pristine"_ (vierge) jusqu'à ce que l'utilisateur modifie sa valeur, et _"dirty"_ (modifié) après que la valeur a été changée. Vous pouvez vérifier ces états via les drapeaux `isTouched`, `isPristine` et `isDirty`, comme vu ci-dessous.

```js
const { isTouched, isPristine, isDirty } = field.state.meta
```

![États d'un champ](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## API de champ

L'API de champ est un objet fourni par un slot scopé utilisant la directive `v-slot`. Ce slot reçoit un argument nommé `field` qui fournit des méthodes et propriétés pour travailler avec l'état du champ.

Exemple :

```vue
<template v-slot="{ field }">
  <input
    :name="field.name"
    :value="field.state.value"
    @blur="field.handleBlur"
    @input="(e) => field.handleChange(e.target.value)"
  />
</template>
```

## Validation

`@tanstack/vue-form` fournit à la fois une validation synchrone et asynchrone prête à l'emploi. Les fonctions de validation peuvent être passées au composant `form.Field` en utilisant la prop `validators`.

Exemple :

```vue
<template>
    <!-- ... -->
    <form.Field
        name="firstName"
        :validators="{
            onChange: ({ value }) =>
                !value
                    ? `Un prénom est requis`
                    : value.length < 3
                        ? `Le prénom doit comporter au moins 3 caractères`
                        : undefined,
            onChangeAsync: async ({ value }) => {
                await new Promise((resolve) => setTimeout(resolve, 1000))
                return value.includes('error') && 'Le prénom ne peut pas contenir "error"'
        },
    }"
    >
        <template v-slot="{ field }">
            <input
                :value="field.state.value"
                @input="(e) => field.handleChange(e.target.value)"
                @blur="field.handleBlur"
            />
            <FieldInfo :field="field" />
        </template>
    </form.Field>
    <!-- ... -->
</template>
```

## Validation avec des bibliothèques de schéma standard

En plus des options de validation personnalisées, nous prenons également en charge la spécification [Standard Schema](https://github.com/standard-schema/standard-schema).

Vous pouvez définir un schéma en utilisant n'importe quelle bibliothèque implémentant la spécification et le passer à un validateur de formulaire ou de champ.

Les bibliothèques prises en charge incluent :

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'
import { z } from 'zod'

const form = useForm({
  // ...
})

const onChangeFirstName = z.string().refine(
  async (value) => {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return !value.includes('error')
  },
  {
    message: "Le prénom ne peut pas contenir 'error'",
  },
)
</script>

<template>
  <!-- ... -->
  <form.Field
    name="firstName"
    :validators="{
      onChange: z
        .string()
        .min(3, 'Le prénom doit comporter au moins 3 caractères'),
      onChangeAsyncDebounceMs: 500,
      onChangeAsync: onChangeFirstName,
    }"
  >
    <template v-slot="{ field, state }">
      <label :htmlFor="field.name">Prénom :</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).value)"
        @blur="field.handleBlur"
      />
      <FieldInfo :state="state" />
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## Réactivité

`@tanstack/vue-form` offre diverses manières de s'abonner aux changements d'état du formulaire et des champs, notamment la méthode `form.useStore` et le composant `form.Subscribe`. Ces méthodes vous permettent d'optimiser les performances de rendu de votre formulaire en ne mettant à jour les composants que lorsque nécessaire.

Exemple :

```vue
<script setup lang="ts">
// ...
const firstName = form.useStore((state) => state.values.firstName)
</script>

<template>
  <!-- ... -->
  <form.Subscribe>
    <template v-slot="{ canSubmit, isSubmitting }">
      <button type="submit" :disabled="!canSubmit">
        {{ isSubmitting ? '...' : 'Soumettre' }}
      </button>
    </template>
  </form.Subscribe>
  <!-- ... -->
</template>
```

## Écouteurs

`@tanstack/vue-form` vous permet de réagir à des déclencheurs spécifiques et de les "écouter" pour déclencher des effets secondaires.

Exemple :

```vue
<template>
  <form.Field
    name="country"
    :listeners="{
      onChange: ({ value }) => {
        console.log(
          `Pays changé en : ${value}, réinitialisation de la province`,
        )
        form.setFieldValue('province', '')
      },
    }"
  >
    <template v-slot="{ field }">
      <input
        :value="field.state.value"
        @input="(e) => field.handleChange(e.target.value)"
      />
    </template>
  </form.Field>
</template>
```

Plus d'informations peuvent être trouvées sur [Écouteurs](./listeners.md)

Note : L'utilisation de la méthode `form.useField` pour obtenir de la réactivité est déconseillée car elle est conçue pour être utilisée avec précaution dans le composant `form.Field`. Vous pourriez préférer utiliser `form.useStore` à la place.

## Champs de tableau

Les champs de tableau vous permettent de gérer une liste de valeurs dans un formulaire, comme une liste de hobbies. Vous pouvez créer un champ de tableau en utilisant le composant `form.Field` avec la prop `mode="array"`.

Lorsque vous travaillez avec des champs de tableau, vous pouvez utiliser les méthodes `pushValue`, `removeValue`, `swapValues` et `moveValue` des champs pour ajouter, supprimer et échanger des valeurs dans le tableau.

Exemple :

```vue
<template>
  <form @submit.prevent.stop="form.handleSubmit">
    <form.Field name="hobbies" mode="array">
      <template v-slot="{ field: hobbiesField }">
        <div>
          Hobbies
          <div>
            <div
              v-if="
                Array.isArray(hobbiesField.state.value) &&
                !hobbiesField.state.value.length
              "
            >
              Aucun hobby trouvé.
            </div>
            <div v-else>
              <div v-for="(_, i) in hobbiesField.state.value" :key="i">
                <form.Field :name="`hobbies[${i}].name`">
                  <template v-slot="{ field }">
                    <div>
                      <label :for="field.name">Nom :</label>
                      <input
                        :id="field.name"
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="(e) => field.handleChange(e.target.value)"
                      />
                      <button
                        type="button"
                        @click="hobbiesField.removeValue(i)"
                      >
                        X
                      </button>
                      <FieldInfo :field="field" />
                    </div>
                  </template>
                </form.Field>
                <form.Field :name="`hobbies[${i}].description`">
                  <template v-slot="{ field }">
                    <div>
                      <label :for="field.name">Description :</label>
                      <input
                        :id="field.name"
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="(e) => field.handleChange(e.target.value)"
                      />
                      <FieldInfo :field="field" />
                    </div>
                  </template>
                </form.Field>
              </div>
            </div>
            <button
              type="button"
              @click="
                hobbiesField.pushValue({
                  name: '',
                  description: '',
                  yearsOfExperience: 0,
                })
              "
            >
              Ajouter un hobby
            </button>
          </div>
        </div>
      </template>
    </form.Field>
  </form>
</template>
```

Voici les concepts de base et la terminologie utilisés dans la bibliothèque `@tanstack/vue-form`. Comprendre ces concepts vous aidera à travailler plus efficacement avec la bibliothèque et à créer des formulaires complexes avec facilité.
