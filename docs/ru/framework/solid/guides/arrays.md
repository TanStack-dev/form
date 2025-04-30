---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T22:01:32.482Z'
id: arrays
title: Массивы
---

TanStack Form поддерживает массивы в качестве значений формы, включая вложенные объекты внутри массива.

## Базовое использование

Для работы с массивом можно использовать `field.state.value` для массива в сочетании с [`Index` из `solid-js`](https://www.solidjs.com/tutorial/flow_index):

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
          {/* Не заменяйте на `For`, иначе поведение будет неожиданным */}
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

> Важно использовать `Index` из `solid-js`, а не `For`, так как `For` вызывает перерендеринг внутренних компонентов при каждом изменении массива.  
> Это приводит к потере значения поля и, как следствие, удалению значения вложенного поля.

Этот код будет генерировать JSX при каждом вызове `pushValue` для `field`:

```jsx
<button onClick={() => field().pushValue({ name: '', age: 0 })} type="button">
  Добавить человека
</button>
```

Для работы с вложенным полем можно использовать следующий синтаксис:

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

## Полный пример

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
                {/* Не заменяйте на For, иначе тест не пройдет */}
                <Index each={field().state.value}>
                  {(_, i) => (
                    <form.Field name={`people[${i}].name`}>
                      {(subField) => (
                        <div>
                          <label>
                            <div>Имя человека {i}</div>
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
                Добавить человека
              </button>
            </div>
          )}
        </form.Field>
        <button type="submit">Отправить</button>
      </form>
    </div>
  )
}
```
