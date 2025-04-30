---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T21:37:07.974Z'
id: form-validation
title: Validation de formulaire
---

Au cœur des fonctionnalités de TanStack Form se trouve le concept de validation. TanStack Form rend la validation hautement personnalisable :

- Vous pouvez contrôler quand effectuer la validation (à chaque changement, à la saisie, lors de la perte de focus, à la soumission...)
- Les règles de validation peuvent être définies au niveau du champ ou au niveau du formulaire
- La validation peut être synchrone ou asynchrone (par exemple, suite à un appel API)

## Quand la validation est-elle effectuée ?

C'est à vous de décider ! Le composant `<Field />` accepte certaines fonctions de rappel (callbacks) comme props, telles que `onChange` ou `onBlur`. Ces fonctions de rappel reçoivent la valeur actuelle du champ ainsi que l'objet fieldAPI, afin que vous puissiez effectuer la validation. Si vous trouvez une erreur de validation, retournez simplement le message d'erreur sous forme de chaîne de caractères, et il sera disponible dans `field.state.meta.errors`.

Voici un exemple :

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

Dans l'exemple ci-dessus, la validation est effectuée à chaque frappe (`onChange`). Si, à la place, nous voulions que la validation soit effectuée lors de la perte de focus du champ, nous modifierions le code comme suit :

```tsx
<form.Field
  name="age"
  validators={{
    onBlur: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        // Écouter l'événement onBlur sur le champ
        onBlur={field.handleBlur}
        // Nous devons toujours implémenter onChange pour que TanStack Form reçoive les modifications
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

Ainsi, vous pouvez contrôler quand la validation est effectuée en implémentant la fonction de rappel souhaitée. Vous pouvez même effectuer différentes validations à différents moments :

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
    onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        // Écouter l'événement onBlur sur le champ
        onBlur={field.handleBlur}
        // Nous devons toujours implémenter onChange pour que TanStack Form reçoive les modifications
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

Dans l'exemple ci-dessus, nous validons différentes choses sur le même champ à différents moments (à chaque frappe et lors de la perte de focus du champ). Comme `field.state.meta.errors` est un tableau, toutes les erreurs pertinentes à un moment donné sont affichées. Vous pouvez également utiliser `field.state.meta.errorMap` pour obtenir les erreurs en fonction de _quand_ la validation a été effectuée (onChange, onBlur, etc.). Plus d'informations sur l'affichage des erreurs ci-dessous.

## Affichage des erreurs

Une fois votre validation en place, vous pouvez mapper les erreurs depuis un tableau pour les afficher dans votre interface :

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => {
    return (
      <>
        {/* ... */}
        {!field.state.meta.isValid && (
          <em>{field.state.meta.errors.join(',')}</em>
        )}
      </>
    )
  }}
</form.Field>
```

Ou utiliser la propriété `errorMap` pour accéder à une erreur spécifique :

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => (
    <>
      {/* ... */}
      {field.state.meta.errorMap['onChange'] ? (
        <em>{field.state.meta.errorMap['onChange']}</em>
      ) : null}
    </>
  )}
</form.Field>
```

Il est important de mentionner que notre tableau `errors` et le `errorMap` correspondent aux types retournés par les validateurs. Cela signifie que :

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }}
>
  {(field) => (
    <>
      {/* ... */}
      {/* errorMap.onChange est de type `{isOldEnough: false} | undefined` */}
      {/* meta.errors est de type `Array<{isOldEnough: false} | undefined>` */}
      {!field.state.meta.errorMap['onChange']?.isOldEnough ? (
        <em>The user is not old enough</em>
      ) : null}
    </>
  )}
</form.Field>
```

## Validation au niveau du champ vs au niveau du formulaire

Comme montré ci-dessus, chaque `<Field>` accepte ses propres règles de validation via les fonctions de rappel `onChange`, `onBlur`, etc. Il est également possible de définir des règles de validation au niveau du formulaire (plutôt que champ par champ) en passant des fonctions de rappel similaires au hook `useForm()`.

Exemple :

```tsx
export default function App() {
  const form = useForm({
    defaultValues: {
      age: 0,
    },
    onSubmit: async ({ value }) => {
      console.log(value)
    },
    validators: {
      // Ajoutez des validateurs au formulaire de la même manière que vous le feriez pour un champ
      onChange({ value }) {
        if (value.age < 13) {
          return 'Must be 13 or older to sign'
        }
        return undefined
      },
    },
  })

  // Abonnez-vous à la map d'erreurs du formulaire pour que les mises à jour soient rendues
  // alternativement, vous pouvez utiliser `form.Subscribe`
  const formErrorMap = useStore(form.store, (state) => state.errorMap)

  return (
    <div>
      {/* ... */}
      {formErrorMap.onChange ? (
        <div>
          <em>There was an error on the form: {formErrorMap.onChange}</em>
        </div>
      ) : null}
      {/* ... */}
    </div>
  )
}
```

### Définir des erreurs au niveau des champs depuis les validateurs du formulaire

Vous pouvez définir des erreurs sur les champs depuis les validateurs du formulaire. Un cas d'usage courant pour cela est de valider tous les champs lors de la soumission en appelant un seul point d'API dans le validateur `onSubmitAsync` du formulaire.

```tsx
export default function App() {
  const form = useForm({
    defaultValues: {
      age: 0,
      socials: [],
      details: {
        email: '',
      },
    },
    validators: {
      onSubmitAsync: async ({ value }) => {
        // Validez la valeur sur le serveur
        const hasErrors = await verifyDataOnServer(value)
        if (hasErrors) {
          return {
            form: 'Invalid data', // La clé `form` est optionnelle
            fields: {
              age: 'Must be 13 or older to sign',
              // Définissez des erreurs sur des champs imbriqués avec le nom du champ
              'socials[0].url': 'The provided URL does not exist',
              'details.email': 'An email is required',
            },
          }
        }

        return null
      },
    },
  })

  return (
    <div>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          void form.handleSubmit()
        }}
      >
        <form.Field name="age">
          {(field) => (
            <>
              <label htmlFor={field.name}>Age:</label>
              <input
                id={field.name}
                name={field.name}
                value={field.state.value}
                type="number"
                onChange={(e) => field.handleChange(e.target.valueAsNumber)}
              />
              {!field.state.meta.isValid && (
                <em role="alert">{field.state.meta.errors.join(', ')}</em>
              )}
            </>
          )}
        </form.Field>
        <form.Subscribe
          selector={(state) => [state.errorMap]}
          children={([errorMap]) =>
            errorMap.onSubmit ? (
              <div>
                <em>There was an error on the form: {errorMap.onSubmit}</em>
              </div>
            ) : null
          }
        />
        {/*...*/}
      </form>
    </div>
  )
}
```

> Il est important de mentionner que si vous avez une fonction de validation de formulaire qui retourne une erreur, cette erreur peut être écrasée par la validation spécifique au champ.
>
> Cela signifie que :
>
> ```jsx
> const form = useForm({
>   defaultValues: {
>     age: 0,
>   },
>   validators: {
>     onChange: ({ value }) => {
>       return {
>         fields: {
>           age: value.age < 12 ? 'Too young!' : undefined,
>         },
>       }
>     },
>   },
> })
>
> // ...
>
> return (
>   <form.Field
>     name="age"
>     validators={{
>       onChange: ({ value }) => (value % 2 === 0 ? 'Must be odd!' : undefined),
>     }}
>     children={() => <>{/* ... */}</>}
>   />
> )
> ```
>
> Affichera uniquement `'Must be odd!'` même si l'erreur 'Too young!' est retournée par la validation au niveau du formulaire.

## Validation Fonctionnelle Asynchrone

Bien que nous supposions que la plupart des validations seront synchrones, il existe de nombreux cas où un appel réseau ou une autre opération asynchrone serait utile pour valider.

Pour ce faire, nous avons des méthodes dédiées `onChangeAsync`, `onBlurAsync`, et d'autres qui peuvent être utilisées pour valider :

```tsx
<form.Field
  name="age"
  validators={{
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value < 13 ? 'You must be 13 to make an account' : undefined
    },
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

Les validations synchrones et asynchrones peuvent coexister. Par exemple, il est possible de définir à la fois `onBlur` et `onBlurAsync` sur le même champ :

```tsx
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
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

La méthode de validation synchrone (`onBlur`) est exécutée en premier et la méthode asynchrone (`onBlurAsync`) n'est exécutée que si la méthode synchrone (`onBlur`) réussit. Pour changer ce comportement, définissez l'option `asyncAlways` sur `true`, et la méthode asynchrone sera exécutée quel que soit le résultat de la méthode synchrone.

### Anti-rebond (Debouncing) intégré

Bien que les appels asynchrones soient la solution pour valider contre la base de données, exécuter une requête réseau à chaque frappe est un bon moyen de DDOSer votre base de données.

À la place, nous permettons une méthode simple pour anti-rebondir vos appels `async` en ajoutant une seule propriété :

```tsx
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsync: async ({ value }) => {
      // ...
    },
  }}
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

Cela anti-rebondira chaque appel asynchrone avec un délai de 500ms. Vous pouvez même surcharger cette propriété pour chaque validation :

```tsx
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
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

Cela exécutera `onChangeAsync` toutes les 1500ms tandis que `onBlurAsync` s'exécutera toutes les 500ms.

## Validation via les bibliothèques de schémas

Bien que les fonctions offrent plus de flexibilité et de personnalisation pour votre validation, elles peuvent être un peu verbeuses. Pour aider à résoudre ce problème, il existe des bibliothèques qui fournissent une validation basée sur des schémas pour rendre la validation concise et strictement typée beaucoup plus facile. Vous pouvez également définir un seul schéma pour votre formulaire entier et le passer au niveau du formulaire, les erreurs seront automatiquement propagées aux champs.

### Bibliothèques de schémas standard

TanStack Form prend nativement en charge toutes les bibliothèques conformes à la [spécification Standard Schema](https://github.com/standard-schema/standard-schema), notamment :

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)
- [Effect/Schema](https://effect.website/docs/schema/standard-schema/)

_Note :_ assurez-vous d'utiliser la dernière version des bibliothèques de schémas car les versions plus anciennes pourraient ne pas encore supporter Standard Schema.

> La validation ne vous fournira pas les valeurs transformées. Voir [gestion de la soumission](./submission-handling.md) pour plus d'informations.

Pour utiliser les schémas de ces bibliothèques, vous pouvez les passer aux props `validators` comme vous le feriez avec une fonction personnalisée :

```tsx
const userSchema = z.object({
  age: z.number().gte(13, 'You must be 13 to make an account'),
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

Les validations asynchrones aux niveaux du formulaire et du champ sont également supportées :

```tsx
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
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

Si vous avez besoin de plus de contrôle sur votre validation Standard Schema, vous pouvez combiner un Standard Schema avec une fonction de rappel comme ceci :

```tsx
<form.Field
  name="age"
  asyncDeb
```
