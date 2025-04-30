---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:49:54.129Z'
id: arrays
title: Arreglos
---

# Arrays

TanStack Form admite arrays como valores en un formulario, incluyendo valores de sub-objetos dentro de un array.

## Uso básico

Para usar un array, puedes utilizar `field.state.value` en un valor de array junto con [`Index` de `solid-js`](https://www.solidjs.com/tutorial/flow_index):

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
          {/* No cambies esto a `For` o las cosas no funcionarán como se espera */}
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

> Debes usar `Index` de `solid-js` y no `For` porque `For` hará que los componentes internos se vuelvan a renderizar
> cada vez que el array cambie.
>
> Esto causa que el campo pierda su valor y por lo tanto elimine el valor del subcampo.

Esto generará el JSX mapeado cada vez que ejecutes `pushValue` en `field`:

```jsx
<button onClick={() => field().pushValue({ name: '', age: 0 })} type="button">
  Añadir persona
</button>
```

Finalmente, puedes usar un subcampo de la siguiente manera:

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

## Ejemplo completo

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
                {/* No cambies esto a For o el test fallará */}
                <Index each={field().state.value}>
                  {(_, i) => (
                    <form.Field name={`people[${i}].name`}>
                      {(subField) => (
                        <div>
                          <label>
                            <div>Nombre para la persona {i}</div>
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
                Añadir persona
              </button>
            </div>
          )}
        </form.Field>
        <button type="submit">Enviar</button>
      </form>
    </div>
  )
}
```
