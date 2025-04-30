---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:35:10.555Z'
id: arrays
title: Tableaux
---

# Tableaux (Arrays)

TanStack Form prend en charge les tableaux comme valeurs dans un formulaire, y compris les valeurs de sous-objets à l'intérieur d'un tableau.

## Utilisation de base

Pour utiliser un tableau, vous pouvez utiliser `field.state.value` sur une valeur de tableau :

```jsx
function App() {
  const form = useForm({
    defaultValues: {
      people: [],
    },
  })

  return (
    <form.Field name="people" mode="array">
      {(field) => (
        <div>
          {field.state.value.map((_, i) => {
            // ...
          })}
        </div>
      )}
    </form.Field>
  )
}
```

Cela générera le JSX mappé chaque fois que vous exécuterez `pushValue` sur `field` :

```jsx
<button onClick={() => field.pushValue({ name: '', age: 0 })} type="button">
  Ajouter une personne
</button>
```

Enfin, vous pouvez utiliser un sous-champ comme ceci :

```jsx
<form.Field key={i} name={`people[${i}].name`}>
  {(subField) => (
    <input
      value={subField.state.value}
      onChange={(e) => subField.handleChange(e.target.value)}
    />
  )}
</form.Field>
```

## Exemple complet

```jsx
function App() {
  const form = useForm({
    defaultValues: {
      people: [],
    },
    onSubmit({ value }) {
      alert(JSON.stringify(value))
    },
  })

  return (
    <div>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          form.handleSubmit()
        }}
      >
        <form.Field name="people" mode="array">
          {(field) => {
            return (
              <div>
                {field.state.value.map((_, i) => {
                  return (
                    <form.Field key={i} name={`people[${i}].name`}>
                      {(subField) => {
                        return (
                          <div>
                            <label>
                              <div>Nom pour la personne {i}</div>
                              <input
                                value={subField.state.value}
                                onChange={(e) =>
                                  subField.handleChange(e.target.value)
                                }
                              />
                            </label>
                          </div>
                        )
                      }}
                    </form.Field>
                  )
                })}
                <button
                  onClick={() => field.pushValue({ name: '', age: 0 })}
                  type="button"
                >
                  Ajouter une personne
                </button>
              </div>
            )
          }}
        </form.Field>
        <form.Subscribe
          selector={(state) => [state.canSubmit, state.isSubmitting]}
          children={([canSubmit, isSubmitting]) => (
            <button type="submit" disabled={!canSubmit}>
              {isSubmitting ? '...' : 'Soumettre'}
            </button>
          )}
        />
      </form>
    </div>
  )
}
```
