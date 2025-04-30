---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:37:36.455Z'
id: arrays
title: Tableaux
---

# Tableaux (Arrays)

TanStack Form prend en charge les tableaux comme valeurs dans un formulaire, y compris les sous-objets à l'intérieur d'un tableau.

## Utilisation de base

Pour utiliser un tableau, vous pouvez utiliser `field.state.value` sur une valeur de tableau en conjonction avec [`Index` de `solid-js`](https://www.solidjs.com/tutorial/flow_index) :

```jsx
function App() {
  const form = createForm(() => ({
    defaultValues: {
      people: [],
    },
  }))

  return (
    <form.Field name="people" mode="array">
      {(field) => (
        <Show when={field().state.value.length > 0}>
          {/* Ne pas remplacer par `For` sinon cela ne fonctionnera pas comme prévu */}
          <Index each={field().state.value}>
            {
              (_, i) => null // ...
            }
          </Index>
        </Show>
      )}
    </form.Field>
  )
}
```

> Vous devez utiliser `Index` de `solid-js` et non `For` car `For` entraînera un nouveau rendu des composants internes
> à chaque modification du tableau.
>
> Cela provoquerait la perte de la valeur du champ et donc la suppression de la valeur du sous-champ.

Ceci générera le JSX mappé chaque fois que vous utiliserez `pushValue` sur `field` :

```jsx
<button onClick={() => field().pushValue({ name: '', age: 0 })} type="button">
  Ajouter une personne
</button>
```

Enfin, vous pouvez utiliser un sous-champ comme ceci :

```jsx
<form.Field name={`people[${i}].name`}>
  {(subField) => (
    <input
      value={subField().state.value}
      onInput={(e) => {
        subField().handleChange(e.currentTarget.value)
      }}
    />
  )}
</form.Field>
```

## Exemple complet

```jsx
function App() {
  const form = createForm(() => ({
    defaultValues: {
      people: [],
    },
    onSubmit: ({ value }) => alert(JSON.stringify(value)),
  }))

  return (
    <div>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          form.handleSubmit()
        }}
      >
        <form.Field name="people">
          {(field) => (
            <div>
              <Show when={field().state.value.length > 0}>
                {/* Ne pas remplacer par For sinon le test échouera */}
                <Index each={field().state.value}>
                  {(_, i) => (
                    <form.Field name={`people[${i}].name`}>
                      {(subField) => (
                        <div>
                          <label>
                            <div>Nom pour la personne {i}</div>
                            <input
                              value={subField().state.value}
                              onInput={(e) => {
                                subField().handleChange(e.currentTarget.value)
                              }}
                            />
                          </label>
                        </div>
                      )}
                    </form.Field>
                  )}
                </Index>
              </Show>

              <button
                onClick={() => field().pushValue({ name: '', age: 0 })}
                type="button"
              >
                Ajouter une personne
              </button>
            </div>
          )}
        </form.Field>
        <button type="submit">Soumettre</button>
      </form>
    </div>
  )
}
```
