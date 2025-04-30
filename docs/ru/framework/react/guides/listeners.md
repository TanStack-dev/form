---
source-updated-at: '2025-04-27T15:54:55.000Z'
translation-updated-at: '2025-04-30T21:59:38.042Z'
id: listeners
title: Слушатели
---

Для ситуаций, когда нужно "воздействовать" или "реагировать" на триггеры, существует API слушателей (listener API). Например, если разработчик хочет сбросить поле формы в результате изменения другого поля, следует использовать API слушателей.

Представьте следующий пользовательский сценарий:

- Пользователь выбирает страну из выпадающего списка.
- Затем пользователь выбирает регион из другого выпадающего списка.
- Пользователь изменяет выбранную страну на другую.

В этом примере при изменении страны выбранный регион необходимо сбросить, так как он больше недействителен. С помощью API слушателей можно подписаться на событие `onChange` и сбросить поле "province" при срабатывании слушателя.

События, на которые можно подписаться:

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

### Встроенный дебаунсинг (debouncing)

Если внутри слушателя выполняется API-запрос, может потребоваться дебаунсинг вызовов, чтобы избежать проблем с производительностью.  
Для этого можно легко добавить задержку с помощью `onChangeDebounceMs` или `onBlurDebounceMs`.

```tsx
<form.Field
  name="country"
  listeners={{
    onChangeDebounceMs: 500, // задержка 500 мс
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

### Слушатели на уровне формы

На более высоком уровне слушатели также доступны для всей формы, что позволяет обрабатывать события `onMount` и `onSubmit`, а также распространять `onChange` и `onBlur` на все дочерние элементы формы. Слушатели на уровне формы также поддерживают дебаунсинг, как описано выше.

Слушатели `onMount` и `onSubmit` имеют доступ к следующим свойствам:

- `formApi`

Слушатели `onChange` и `onBlur` имеют доступ к:

- `fieldApi`
- `formApi`

```tsx
const form = useForm({
  listeners: {
    onMount: ({ formApi }) => {
      // пользовательский сервис логирования
      loggingService('mount', formApi.state.values)
    },

    onChange: ({ formApi, fieldApi }) => {
      // логика автосохранения
      if (formApi.state.isValid) {
        formApi.handleSubmit()
      }

      // fieldApi представляет поле, вызвавшее событие
      console.log(fieldApi.name, fieldApi.state.value)
    },
    onChangeDebounceMs: 500,
  },
})
```
