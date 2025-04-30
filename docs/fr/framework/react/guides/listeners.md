---
source-updated-at: '2025-04-27T15:54:55.000Z'
translation-updated-at: '2025-04-30T21:35:36.987Z'
id: listeners
title: Écouteurs
---

Pour les situations où vous souhaitez "affecter" ou "réagir" à des déclencheurs, il existe l'API d'écoute (listener API). Par exemple, si vous, en tant que développeur, souhaitez réinitialiser un champ de formulaire suite à la modification d'un autre champ, vous utiliseriez l'API d'écoute.

Imaginez le flux utilisateur suivant :

- L'utilisateur sélectionne un pays dans une liste déroulante.
- L'utilisateur sélectionne ensuite une province dans une autre liste déroulante.
- L'utilisateur change le pays sélectionné pour un autre.

Dans cet exemple, lorsque l'utilisateur change le pays, la province sélectionnée doit être réinitialisée car elle n'est plus valide. Avec l'API d'écoute, nous pouvons nous abonner à l'événement `onChange` et déclencher une réinitialisation du champ "province" lorsque l'écouteur est activé.

Les événements qui peuvent être "écoutés" sont :

- `onChange`
- `onBlur`
- `onMount`
- `onSubmit`

```tsx
function App() {
  const form = useForm({
    defaultValues: {
      country: '',
      province: '',
    },
    // ...
  })

  return (
    <div>
      <form.Field
        name="country"
        listeners={{
          onChange: ({ value }) => {
            console.log(`Country changed to: ${value}, resetting province`)
            form.setFieldValue('province', '')
          },
        }}
      >
        {(field) => (
          <label>
            <div>Country</div>
            <input
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
          </label>
        )}
      </form.Field>

      <form.Field name="province">
        {(field) => (
          <label>
            <div>Province</div>
            <input
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
          </label>
        )}
      </form.Field>
    </div>
  )
}
```

### Anti-rebonds intégré (Debouncing)

Si vous effectuez une requête API dans un écouteur, vous pouvez souhaiter appliquer un anti-rebonds (debounce) aux appels pour éviter des problèmes de performance.
Nous proposons une méthode simple pour appliquer un anti-rebonds à vos écouteurs en ajoutant `onChangeDebounceMs` ou `onBlurDebounceMs`.

```tsx
<form.Field
  name="country"
  listeners={{
    onChangeDebounceMs: 500, // anti-rebonds de 500ms
    onChange: ({ value }) => {
      console.log(`Country changed to: ${value} without a change within 500ms, resetting province`)
      form.setFieldValue('province', '')
    },
  }}
>
  {(field) => (
    /* ... */
  )}
</form.Field>
```

### Écouteurs de formulaire

À un niveau supérieur, les écouteurs sont également disponibles au niveau du formulaire, vous donnant accès aux événements `onMount` et `onSubmit`, et permettant à `onChange` et `onBlur` de se propager à tous les enfants du formulaire. Les écouteurs au niveau du formulaire peuvent également bénéficier d'un anti-rebonds de la même manière que précédemment.

Les écouteurs `onMount` et `onSubmit` ont accès aux propriétés suivantes :

- `formApi`

Les écouteurs `onChange` et `onBlur` ont accès à :

- `fieldApi`
- `formApi`

```tsx
const form = useForm({
  listeners: {
    onMount: ({ formApi }) => {
      // service de journalisation personnalisé
      loggingService('mount', formApi.state.values)
    },

    onChange: ({ formApi, fieldApi }) => {
      // logique de sauvegarde automatique
      if (formApi.state.isValid) {
        formApi.handleSubmit()
      }

      // fieldApi représente le champ qui a déclenché l'événement.
      console.log(fieldApi.name, fieldApi.state.value)
    },
    onChangeDebounceMs: 500,
  },
})
```
