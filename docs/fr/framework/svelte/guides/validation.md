---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:39:43.784Z'
id: form-validation
title: Validation de formulaire
---

Au cœur des fonctionnalités de TanStack Form se trouve le concept de validation. TanStack Form rend la validation hautement personnalisable :

- Vous pouvez contrôler quand effectuer la validation (à chaque changement, à la saisie, lors de la perte de focus, à la soumission...)
- Les règles de validation peuvent être définies au niveau du champ ou au niveau du formulaire
- La validation peut être synchrone ou asynchrone (par exemple, suite à un appel API)

## Quand la validation est-elle effectuée ?

C'est à vous de décider ! Le composant `<form.Field />` accepte certaines fonctions de rappel (callbacks) comme props, telles que `onChange` ou `onBlur`. Ces fonctions reçoivent la valeur actuelle du champ ainsi que l'objet fieldAPI, vous permettant ainsi d'effectuer la validation. Si vous détectez une erreur de validation, renvoyez simplement le message d'erreur sous forme de chaîne de caractères, et il sera disponible dans `field.state.meta.errors`.

Voici un exemple :

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

Dans l'exemple ci-dessus, la validation est effectuée à chaque frappe (`onchange`). Si, à la place, nous souhaitions que la validation soit effectuée lors de la perte de focus (blur), nous modifierions le code comme suit :

```svelte
<form.Field
  name="age"
  validators={{
    onBlur: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

Ainsi, vous pouvez contrôler quand la validation est effectuée en implémentant la fonction de rappel souhaitée. Vous pouvez même effectuer différentes validations à différents moments :

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
    onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

Dans l'exemple ci-dessus, nous validons différentes choses sur le même champ à différents moments (à chaque frappe et lors de la perte de focus). Comme `field.state.meta.errors` est un tableau, toutes les erreurs pertinentes à un moment donné sont affichées. Vous pouvez également utiliser `field.state.meta.errorMap` pour obtenir les erreurs en fonction du moment où la validation a été effectuée (onChange, onBlur, etc.). Plus d'informations sur l'affichage des erreurs ci-dessous.

## Affichage des erreurs

Une fois votre validation en place, vous pouvez mapper les erreurs depuis un tableau pour les afficher dans votre interface :

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

Ou utiliser la propriété `errorMap` pour accéder à une erreur spécifique :

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    {#if field.state.meta.errorMap['onChange']}
      <em role="alert">{field.state.meta.errorMap['onChange']}</em>
    {/if}
  {/snippet}
</form.Field>
```

Il est important de noter que notre tableau `errors` et le `errorMap` correspondent aux types renvoyés par les validateurs. Cela signifie que :

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    <!-- errorMap.onChange est de type `{isOldEnough: false} | undefined` -->
    <!-- meta.errors est de type `Array<{isOldEnough: false} | undefined>` -->
    {#if field.state.meta.errorMap['onChange']?.isOldEnough}
        <em>The user is not old enough</em>
    {/if}
  {/snippet}
</form.Field>
```

## Validation au niveau du champ vs au niveau du formulaire

Comme montré ci-dessus, chaque `<form.Field>` accepte ses propres règles de validation via les fonctions de rappel `onChange`, `onBlur`, etc. Il est également possible de définir des règles de validation au niveau du formulaire (plutôt que champ par champ) en passant des fonctions de rappel similaires au hook `createForm()`.

Exemple :

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      age: 0,
    },
    onSubmit: async ({ value }) => {
      console.log(value)
    },
    validators: {
      // Ajoutez des validateurs au formulaire de la même manière que pour un champ
      onChange({ value }) {
        if (value.age < 13) {
          return 'Must be 13 or older to sign'
        }
        return undefined
      },
    },
  }))

  // Abonnez-vous à la map d'erreurs du formulaire pour que les mises à jour s'affichent
  // alternativement, vous pouvez utiliser `form.Subscribe`
  const formErrorMap = form.useStore((state) => state.errorMap)
</script>

<div>
  <!-- ... -->
  {#if formErrorMap.current.onChange}
    <div>
      <em>There was an error on the form: {formErrorMap.current.onChange}</em>
    </div>
  {/if}
  <!-- ... -->
</div>
```

## Validation fonctionnelle asynchrone

Bien que nous supposions que la plupart des validations seront synchrones, il existe de nombreux cas où un appel réseau ou une autre opération asynchrone serait utile pour valider.

Pour cela, nous avons des méthodes dédiées `onChangeAsync`, `onBlurAsync` et d'autres qui peuvent être utilisées pour valider :

```svelte
<form.Field
  name="age"
  validators={{
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value < 13 ? 'You must be 13 to make an account' : undefined
    },
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

Les validations synchrones et asynchrones peuvent coexister. Par exemple, il est possible de définir à la fois `onBlur` et `onBlurAsync` sur le même champ :

```svelte
<form.Field
  name="age"
  validators={{
    onBlur: ({ value }) => (value < 13 ? 'You must be at least 13' : undefined),
    onBlurAsync: async ({ value }) => {
      const currentAge = await fetchCurrentAgeOnProfile()
      return value < currentAge ? 'You can only increase the age' : undefined
    },
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

La méthode de validation synchrone (`onBlur`) est exécutée en premier, et la méthode asynchrone (`onBlurAsync`) n'est exécutée que si la première réussit. Pour modifier ce comportement, définissez l'option `asyncAlways` sur `true`, et la méthode asynchrone sera exécutée quelle que soit le résultat de la méthode synchrone.

### Anti-rebond (Debouncing) intégré

Bien que les appels asynchrones soient la solution pour valider par rapport à la base de données, exécuter une requête réseau à chaque frappe est un bon moyen de saturer votre base de données.

À la place, nous proposons une méthode simple pour appliquer un anti-rebond (debounce) à vos appels asynchrones en ajoutant une seule propriété :

```svelte
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsync: async ({ value }) => {
      // ...
    },
  }}
>
  <!-- ... -->
</form.Field>
```

Cela appliquera un anti-rebond de 500ms à chaque appel asynchrone. Vous pouvez même surcharger cette propriété pour chaque validation :

```svelte
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsyncDebounceMs: 1500,
    onChangeAsync: async ({ value }) => {
      // ...
    },
    onBlurAsync: async ({ value }) => {
      // ...
    },
  }}
>
  <!-- ... -->
</form.Field>
```

> Cela exécutera `onChangeAsync` toutes les 1500ms tandis que `onBlurAsync` s'exécutera toutes les 500ms.

## Validation via des bibliothèques de schémas

Bien que les fonctions offrent plus de flexibilité et de personnalisation pour votre validation, elles peuvent être un peu verbeuses. Pour résoudre ce problème, il existe des bibliothèques qui fournissent une validation basée sur des schémas, rendant la validation concise et strictement typée beaucoup plus facile. Vous pouvez également définir un seul schéma pour votre formulaire entier et le passer au niveau du formulaire, les erreurs seront automatiquement propagées aux champs.

### Bibliothèques de schémas standard

TanStack Form prend nativement en charge toutes les bibliothèques conformes à la [spécification Standard Schema](https://github.com/standard-schema/standard-schema), notamment :

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_Note :_ assurez-vous d'utiliser la dernière version des bibliothèques de schémas, car les versions plus anciennes pourraient ne pas encore supporter Standard Schema.

Pour utiliser les schémas de ces bibliothèques, vous pouvez les passer aux props `validators` comme vous le feriez avec une fonction personnalisée :

```svelte
<script>
  import { z } from 'zod'

  // ...

  const form = createForm(() => ({
    // ...
  }))
</script>

<form.Field
  name="age"
  validators={{
    onChange: z.number().gte(13, 'You must be 13 to make an account'),
  }}
>
  <!-- ... -->
</form.Field>
```

Les validations asynchrones au niveau du formulaire et des champs sont également prises en charge :

```svelte
<form.Field
  name="age"
  validators={{
    onChange: z.number().gte(13, 'You must be 13 to make an account'),
    onChangeAsyncDebounceMs: 500,
    onChangeAsync: z.number().refine(
      async (value) => {
        const currentAge = await fetchCurrentAgeOnProfile()
        return value >= currentAge
      },
      {
        message: 'You can only increase the age',
      },
    ),
  }}
>
  <!-- ... -->
</form.Field>
```

## Empêcher la soumission de formulaires invalides

Les fonctions de rappel `onChange`, `onBlur`, etc. sont également exécutées lors de la soumission du formulaire, et la soumission est bloquée si le formulaire est invalide.

L'objet d'état du formulaire possède un indicateur `canSubmit` qui est faux lorsque n'importe quel champ est invalide et que le formulaire a été modifié (`canSubmit` reste vrai jusqu'à ce que le formulaire ait été modifié, même si certains champs sont "techniquement" invalides selon leurs props `onChange`/`onBlur`).

Vous pouvez vous y abonner via `form.Subscribe` et utiliser la valeur pour, par exemple, désactiver le bouton de soumission lorsque le formulaire est invalide (en pratique, les boutons désactivés ne sont pas accessibles, utilisez plutôt `aria-disabled`).

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    /* ... */
  }))
</script>

<!-- ... -->

<!-- Bouton de soumission dynamique -->
<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
  children={(state) => (
    <button type="submit" disabled={!state().canSubmit}>
      {state().isSubmitting ? '...' : 'Submit'}
    </button>
  )}
>
  {#snippet children(state)}
    <button type="submit" disabled={!state.canSubmit}>
      {state.isSubmitting ? '...' : 'Submit'}
    </button>
  {/snippet}
</form.Subscribe>
```
