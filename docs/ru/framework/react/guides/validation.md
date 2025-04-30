---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T22:01:07.089Z'
id: form-validation
title: Валидация формы
---

Основой функциональности TanStack Form является концепция валидации. TanStack Form делает валидацию максимально настраиваемой:

- Вы можете контролировать, когда выполнять валидацию (при изменении, вводе, потере фокуса, отправке...)
- Правила валидации можно определять как на уровне поля, так и на уровне формы
- Валидация может быть синхронной или асинхронной (например, в результате вызова API)

## Когда выполняется валидация?

Это зависит от вас! Компонент `<Field />` принимает колбэки в качестве пропсов, такие как `onChange` или `onBlur`. Эти колбэки получают текущее значение поля и объект fieldAPI, чтобы вы могли выполнить валидацию. Если обнаружена ошибка валидации, просто верните сообщение об ошибке в виде строки, и оно будет доступно в `field.state.meta.errors`.

Вот пример:

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

В примере выше валидация выполняется при каждом нажатии клавиши (`onChange`). Если бы мы хотели, чтобы валидация выполнялась при потере фокуса полем, код изменился бы так:

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
        // Слушаем событие onBlur на поле
        onBlur={field.handleBlur}
        // onChange всегда нужно реализовывать, чтобы TanStack Form получал изменения
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

Таким образом, вы можете контролировать момент валидации, реализуя нужный колбэк. Можно даже выполнять разные проверки в разное время:

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
        // Слушаем событие onBlur на поле
        onBlur={field.handleBlur}
        // onChange всегда нужно реализовывать, чтобы TanStack Form получал изменения
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

В примере выше мы проверяем разные условия для одного поля в разное время (при каждом нажатии клавиши и при потере фокуса). Поскольку `field.state.meta.errors` является массивом, отображаются все актуальные ошибки на данный момент. Также можно использовать `field.state.meta.errorMap` для получения ошибок в зависимости от момента валидации (onChange, onBlur и т.д.). Подробнее о выводе ошибок ниже.

## Отображение ошибок

После настройки валидации можно преобразовать ошибки из массива для отображения в интерфейсе:

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

Или использовать свойство `errorMap` для доступа к конкретной ошибке:

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

Стоит отметить, что массив `errors` и `errorMap` соответствуют типам, возвращаемым валидаторами. Это означает, что:

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
      {/* errorMap.onChange имеет тип `{isOldEnough: false} | undefined` */}
      {/* meta.errors имеет тип `Array<{isOldEnough: false} | undefined>` */}
      {!field.state.meta.errorMap['onChange']?.isOldEnough ? (
        <em>The user is not old enough</em>
      ) : null}
    </>
  )}
</form.Field>
```

## Валидация на уровне поля vs на уровне формы

Как показано выше, каждый `<Field>` принимает свои правила валидации через колбэки `onChange`, `onBlur` и т.д. Также можно определить правила валидации на уровне формы (в отличие от отдельных полей), передав аналогичные колбэки в хук `useForm()`.

Пример:

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
      // Добавляем валидаторы к форме так же, как к полю
      onChange({ value }) {
        if (value.age < 13) {
          return 'Must be 13 or older to sign'
        }
        return undefined
      },
    },
  })

  // Подписываемся на errorMap формы, чтобы обновления отображались
  // альтернативно можно использовать `form.Subscribe`
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

### Установка ошибок уровня поля из валидаторов формы

Можно устанавливать ошибки для полей из валидаторов формы. Распространённый сценарий — валидация всех полей при отправке формы через вызов одного API-эндпоинта в валидаторе `onSubmitAsync` формы.

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
        // Валидируем данные на сервере
        const hasErrors = await verifyDataOnServer(value)
        if (hasErrors) {
          return {
            form: 'Invalid data', // Ключ `form` опционален
            fields: {
              age: 'Must be 13 or older to sign',
              // Устанавливаем ошибки для вложенных полей по их имени
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

> Важно отметить, что если функция валидации формы возвращает ошибку, эта ошибка может быть перезаписана валидацией конкретного поля.
>
> Это означает, что:
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
> Будет показывать только `'Must be odd!'`, даже если валидация уровня формы вернула ошибку 'Too young!'.

## Асинхронная функциональная валидация

Хотя большинство валидаций, вероятно, будут синхронными, есть много случаев, когда полезно выполнять сетевой запрос или другую асинхронную операцию для валидации.

Для этого предусмотрены специальные методы `onChangeAsync`, `onBlurAsync` и другие:

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

Синхронные и асинхронные валидации могут сосуществовать. Например, можно определить и `onBlur`, и `onBlurAsync` для одного поля:

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

Сначала выполняется синхронный метод валидации (`onBlur`), а асинхронный метод (`onBlurAsync`) выполняется только в случае успешного завершения синхронного. Чтобы изменить это поведение, установите параметр `asyncAlways` в `true`, и асинхронный метод будет выполняться независимо от результата синхронного.

### Встроенный дебаунсинг

Хотя асинхронные запросы — это правильный способ валидации данных в базе, выполнение сетевого запроса при каждом нажатии клавиши — хороший способ DDOS-атаки на вашу базу данных.

Вместо этого мы предлагаем простой метод дебаунсинга асинхронных вызовов с помощью одного свойства:

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

Это установит задержку в 500 мс для каждого асинхронного вызова. Можно даже переопределить это свойство для отдельных валидаций:

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

В этом случае `onChangeAsync` будет выполняться каждые 1500 мс, а `onBlurAsync` — каждые 500 мс.

## Валидация через библиотеки схем

Хотя функции предоставляют больше гибкости и возможностей для настройки валидации, они могут быть многословными. Для решения этой проблемы существуют библиотеки, предлагающие валидацию на основе схем, что значительно упрощает создание кратких и строго типизированных проверок. Также можно определить единую схему для всей формы и передать её на уровне формы, ошибки автоматически распространятся на поля.

### Стандартные библиотеки схем

TanStack Form нативно поддерживает все библиотеки, соответствующие [спецификации Standard Schema](https://github.com/standard-schema/standard-schema), в частности:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)
- [Effect/Schema](https://effect.website/docs/schema/standard-schema/)

_Примечание:_ убедитесь, что используете последнюю версию библиотек схем, так как старые версии могут не поддерживать Standard Schema.

> Валидация не предоставляет преобразованные значения. Подробнее см. в [обработке отправки](./submission-handling.md).

Чтобы использовать схемы из этих библиотек, можно передать их в пропс `validators` так же, как пользовательскую функцию:

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

Асинхронные валидации на уровне формы и полей также поддерживаются:

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

Если требуется ещё больше контроля над валидацией Standard Schema, можно комбинировать Standard Schema с колбэком:

```tsx
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsync: async ({ value, fieldApi }) => {
      const errors = fieldApi.parseValueWithSchema(
        z.number().gte(13, 'You must be 13 to make an account'),
      )
      if (errors) return errors
      // продолжаем валидацию
    },
  }}
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

## Предотвращение отправки невалидных форм

Колбэки `onChange
