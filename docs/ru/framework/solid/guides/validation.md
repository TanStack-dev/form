---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T22:02:51.348Z'
id: form-validation
title: Валидация формы
---

Основой функциональности TanStack Form является концепция валидации. TanStack Form делает валидацию максимально настраиваемой:

- Вы можете контролировать, когда выполнять валидацию (при изменении, при вводе, при потере фокуса, при отправке...)
- Правила валидации можно определять на уровне поля или на уровне формы
- Валидация может быть синхронной или асинхронной (например, в результате вызова API)

## Когда выполняется валидация?

Это зависит от вас! Компонент `<Field />` принимает некоторые колбэки в качестве пропсов, такие как `onChange` или `onBlur`. Эти колбэки получают текущее значение поля, а также объект fieldAPI, чтобы вы могли выполнить валидацию. Если вы обнаружите ошибку валидации, просто верните сообщение об ошибке в виде строки, и оно будет доступно в `field().state.meta.errors`.

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
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {!field().state.meta.isValid ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

В примере выше валидация выполняется при каждом нажатии клавиши (`onChange`). Если бы мы хотели, чтобы валидация выполнялась при потере фокуса полем, мы изменили бы код следующим образом:

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
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        // Слушаем событие onBlur на поле
        onBlur={field().handleBlur}
        // Нам всегда нужно реализовывать onInput, чтобы TanStack Form получал изменения
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {!field().state.meta.isValid ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

Таким образом, вы можете контролировать, когда выполняется валидация, реализуя нужный колбэк. Вы даже можете выполнять разные проверки в разное время:

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
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        // Слушаем событие onBlur на поле
        onBlur={field().handleBlur}
        // Нам всегда нужно реализовывать onInput, чтобы TanStack Form получал изменения
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {!field().state.meta.isValid ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

В примере выше мы проверяем разные условия для одного и того же поля в разное время (при каждом нажатии клавиши и при потере фокуса). Поскольку `field().state.meta.errors` является массивом, все соответствующие ошибки в данный момент времени отображаются. Вы также можете использовать `field().state.meta.errorMap`, чтобы получить ошибки в зависимости от того, _когда_ была выполнена валидация (onChange, onBlur и т.д.). Подробнее о выводе ошибок ниже.

## Вывод ошибок

После настройки валидации вы можете отображать ошибки из массива в вашем интерфейсе:

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
        {!field().state.meta.isValid ? (
          <em>{field().state.meta.errors.join(',')}</em>
        ) : null}
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
      {field().state.meta.errorMap['onChange'] ? (
        <em>{field().state.meta.errorMap['onChange']}</em>
      ) : null}
    </>
  )}
</form.Field>
```

Стоит отметить, что наш массив `errors` и `errorMap` соответствуют типам, возвращаемым валидаторами. Это означает, что:

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
      {!field().state.meta.errorMap['onChange']?.isOldEnough ? (
        <em>The user is not old enough</em>
      ) : null}
    </>
  )}
</form.Field>
```

## Валидация на уровне поля vs на уровне формы

Как показано выше, каждый `<Field>` принимает свои собственные правила валидации через колбэки `onChange`, `onBlur` и т.д. Также можно определить правила валидации на уровне формы (в отличие от поля за полем), передав аналогичные колбэки в хук `createForm()`.

Пример:

```tsx
export default function App() {
  const form = createForm(() => ({
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
  }))

  // Подписываемся на карту ошибок формы, чтобы обновления отображались
  // альтернативно можно использовать `form.Subscribe`
  const formErrorMap = form.useStore((state) => state.errorMap)

  return (
    <div>
      {/* ... */}
      {formErrorMap().onChange ? (
        <div>
          <em>There was an error on the form: {formErrorMap().onChange}</em>
        </div>
      ) : null}
      {/* ... */}
    </div>
  )
}
```

### Установка ошибок на уровне поля из валидаторов формы

Вы можете устанавливать ошибки на полях из валидаторов формы. Один из распространенных случаев использования — валидация всех полей при отправке, вызывая один API-эндпоинт в валидаторе `onSubmitAsync` формы.

```tsx
import { Show } from 'solid-js'
import { createForm } from '@tanstack/solid-form'

export default function App() {
  const form = createForm(() => ({
    defaultValues: {
      age: 0,
      socials: [],
      details: {
        email: '',
      },
    },
    validators: {
      onSubmitAsync: async ({ value }) => {
        // Валидируем значение на сервере
        const hasErrors = await validateDataOnServer(value)
        if (hasErrors) {
          return {
            form: 'Invalid data', // Ключ `form` опционален
            fields: {
              age: 'Must be 13 or older to sign',
              // Устанавливаем ошибки на вложенные поля с помощью имени поля
              'socials[0].url': 'The provided URL does not exist',
              'details.email': 'An email is required',
            },
          }
        }

        return null
      },
    },
  }))

  return (
    <div>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          void form.handleSubmit()
        }}
      >
        <form.Field
          name="age"
          children={(field) => (
            <>
              <label for={field().name}>Age:</label>
              <input
                id={field().name}
                name={field().name}
                value={field().state.value}
                type="number"
                onChange={(e) => field().handleChange(e.target.valueAsNumber)}
              />
              <Show when={field().state.meta.errors.length > 0}>
                <em role="alert">{field().state.meta.errors.join(', ')}</em>
              </Show>
            </>
          )}
        />
        <form.Subscribe
          selector={(state) => ({ errors: state.errors })}
          children={(state) => (
            <Show when={state().errors.length > 0}>
              <div>
                <em>
                  There was an error on the form: {state().errors.join(', ')}
                </em>
              </div>
            </Show>
          )}
        />

        <button type="submit">Submit</button>
        {/*...*/}
      </form>
    </div>
  )
}
```

> Стоит отметить, что если у вас есть функция валидации формы, которая возвращает ошибку, эта ошибка может быть перезаписана валидацией конкретного поля.
>
> Это означает, что:
>
> ```tsx
>  const form = createForm(() => ({
>    defaultValues: {
>      age: 0,
>    },
>    validators: {
>      onChange: ({ value }) => {
>        return {
>          fields: {
>            age: value.age < 12 ? 'Too young!' : undefined,
>          },
>        };
>      },
>    },
>  }));
>
>  return (
>    <form.Field
>      name="age"
>      validators={{
>        onChange: ({ value }) => (value % 2 === 0 ? 'Must be odd!' : undefined),
>      }}
>      children={() => <>{/* ... */}</>}
>    />
>  );
> }
> ```
>
> Будет показывать только `'Must be odd!'`, даже если ошибка 'Too young!' возвращается валидацией на уровне формы.

## Асинхронная функциональная валидация

Хотя мы предполагаем, что большинство валидаций будут синхронными, есть много случаев, когда вызов сети или другая асинхронная операция была бы полезна для валидации.

Для этого у нас есть специальные методы `onChangeAsync`, `onBlurAsync` и другие, которые можно использовать для валидации:

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
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {!field().state.meta.isValid ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

Синхронные и асинхронные валидации могут сосуществовать. Например, можно определить и `onBlur`, и `onBlurAsync` для одного и того же поля:

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
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        onBlur={field().handleBlur}
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {field().state.meta.errors ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

Сначала выполняется синхронный метод валидации (`onBlur`), а асинхронный метод (`onBlurAsync`) выполняется только в случае успешного выполнения синхронного (`onBlur`). Чтобы изменить это поведение, установите опцию `asyncAlways` в `true`, и асинхронный метод будет выполняться независимо от результата синхронного.

### Встроенный дебаунсинг

Хотя асинхронные вызовы — это правильный путь при валидации с базой данных, выполнение сетевого запроса при каждом нажатии клавиши — хороший способ DDoS-атаки на вашу базу данных.

Вместо этого мы предоставляем простой метод для дебаунсинга ваших `async`-вызовов, добавив одно свойство:

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

Это будет дебаунсить каждый асинхронный вызов с задержкой 500 мс. Вы даже можете переопределить это свойство для каждого валидатора:

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

Это будет выполнять `onChangeAsync` каждые 1500 мс, а `onBlurAsync` — каждые 500 мс.

## Валидация через библиотеки схем

Хотя функции предоставляют больше гибкости и настройки для валидации, они могут быть многословными. Чтобы помочь в этом, существуют библиотеки, предоставляющие валидацию на основе схем, что значительно упрощает краткую и строго типизированную валидацию. Вы также можете определить одну схему для всей формы и передать ее на уровень формы, ошибки будут автоматически распространяться на поля.

### Стандартные библиотеки схем

TanStack Form нативно поддерживает все библиотеки, соответствующие [спецификации Standard Schema](https://github.com/standard-schema/standard-schema), в частности:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_Примечание:_ убедитесь, что используете последнюю версию библиотек схем, так как старые версии могут еще не поддерживать Standard Schema.

> Валидация не предоставит вам преобразованные значения. Подробнее см. [обработку отправки](./submission-handling.md).

Чтобы использовать схемы из этих библиотек, вы можете передать их в проп `validators`, как и с пользовательской функцией:

```tsx
import { z } from 'zod'

// ...

const form = createForm(() => ({
  // ...
}))

;<form.Field
  name="age"
  validators={{
    onChange: z.number().gte(13, 'You must be 13 to make an account'),
  }}
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

Асинхронные валидации на уровне формы и поля также поддерживаются:

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

Если вам нужно еще больше контроля над валидацией Standard Schema, вы можете комби
