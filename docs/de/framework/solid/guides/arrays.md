---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:55:44.533Z'
id: arrays
title: Arrays
---

# Arrays

TanStack Form unterstützt Arrays als Werte in einem Formular, einschließlich Unterobjekt-Werten innerhalb eines Arrays.

## Grundlegende Verwendung

Um ein Array zu verwenden, können Sie `field.state.value` auf einem Array-Wert in Kombination mit [`Index` von `solid-js`](https://www.solidjs.com/tutorial/flow_index) nutzen:

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
          {/* Ändern Sie dies nicht zu `For`, sonst funktioniert es nicht wie erwartet */}
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

> Sie müssen `Index` von `solid-js` verwenden und nicht `For`, da `For` dazu führt, dass die inneren Komponenten bei jeder Änderung des Arrays neu gerendert werden.
>
> Dies führt dazu, dass das Feld seinen Wert verliert und somit der Wert des Unterfelds gelöscht wird.

Dies generiert das gemappte JSX jedes Mal, wenn Sie `pushValue` auf `field` ausführen:

```jsx
<button onClick={() => field().pushValue({ name: '', age: 0 })} type="button">
  Person hinzufügen
</button>
```

Schließlich können Sie ein Unterfeld wie folgt verwenden:

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

## Vollständiges Beispiel

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
                {/* Ändern Sie dies nicht zu For, sonst schlägt der Test fehl */}
                <Index each={field().state.value}>
                  {(_, i) => (
                    <form.Field name={`people[${i}].name`}>
                      {(subField) => (
                        <div>
                          <label>
                            <div>Name für Person {i}</div>
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
                Person hinzufügen
              </button>
            </div>
          )}
        </form.Field>
        <button type="submit">Absenden</button>
      </form>
    </div>
  )
}
```
