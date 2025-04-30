---
source-updated-at: '2025-04-16T08:45:06.000Z'
translation-updated-at: '2025-04-30T22:00:19.778Z'
id: basic-concepts
title: Основные концепции
---

Эта страница знакомит с основными концепциями и терминологией, используемыми в библиотеке `@tanstack/react-form`. Ознакомление с этими понятиями поможет вам лучше понять и эффективнее работать с библиотекой.

## Настройки формы (Form Options)

Вы можете создать настройки для своей формы, чтобы их можно было использовать в нескольких формах, с помощью функции `formOptions`.

Пример:

```tsx
interface User {
  firstName: string
  lastName: string
  hobbies: Array<string>
}
const defaultUser: User = { firstName: '', lastName: '', hobbies: [] }

const formOpts = formOptions({
  defaultValues: defaultUser,
})
```

## Экземпляр формы (Form Instance)

Экземпляр формы — это объект, представляющий отдельную форму и предоставляющий методы и свойства для работы с ней. Экземпляр формы создаётся с помощью хука `useForm`, предоставляемого настройками формы. Хук принимает объект с функцией `onSubmit`, которая вызывается при отправке формы.

```tsx
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // Обработка данных формы
    console.log(value)
  },
})
```

Вы также можете создать экземпляр формы без использования `formOptions`, воспользовавшись автономным API `useForm`:

```tsx
interface User {
  firstName: string
  lastName: string
  hobbies: Array<string>
}
const defaultUser: User = { firstName: '', lastName: '', hobbies: [] }

const form = useForm({
  defaultValues: defaultUser,
  onSubmit: async ({ value }) => {
    // Обработка данных формы
    console.log(value)
  },
})
```

## Поле (Field)

Поле представляет собой отдельный элемент ввода формы, например текстовое поле или чекбокс. Поля создаются с помощью компонента `form.Field`, предоставляемого экземпляром формы. Компонент принимает проп `name`, который должен соответствовать ключу в значениях формы по умолчанию. Также он принимает проп `children` — функцию рендера, которая получает объект поля в качестве аргумента.

Пример:

```tsx
<form.Field
  name="firstName"
  children={(field) => (
    <>
      <input
        value={field.state.value}
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.value)}
      />
      <FieldInfo field={field} />
    </>
  )}
/>
```

## Состояние поля (Field State)

Каждое поле имеет собственное состояние, включающее текущее значение, статус валидации, сообщения об ошибках и другие метаданные. Доступ к состоянию поля можно получить через свойство `field.state`.

Пример:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Существует три состояния поля, которые могут быть полезны для отслеживания взаимодействия пользователя с полем: поле считается _"touched"_, когда пользователь кликает/переходит в него, _"pristine"_ до изменения значения и _"dirty"_ после изменения. Эти состояния можно проверить с помощью флагов `isTouched`, `isPristine` и `isDirty`, как показано ниже.

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Состояния поля](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

> **Важное примечание для пользователей `React Hook Form`**: флаг `isDirty` в `TanStack/form` отличается от одноимённого флага в RHF.
> В RHF `isDirty = true`, когда значения формы отличаются от исходных. Если пользователь изменит значения формы, а затем вернёт их к значениям по умолчанию, в RHF `isDirty` будет `false`, а в `TanStack/form` — `true`.
> Значения по умолчанию доступны как на уровне формы, так и на уровне поля в `TanStack/form` (`form.options.defaultValues`, `field.options.defaultValue`), поэтому вы можете написать собственный хелпер `isDefaultValue()`, если нужно эмулировать поведение RHF.

## API поля (Field API)

API поля — это объект, передаваемый в функцию рендера при создании поля. Он предоставляет методы для работы с состоянием поля.

Пример:

```tsx
<input
  value={field.state.value}
  onBlur={field.handleBlur}
  onChange={(e) => field.handleChange(e.target.value)}
/>
```

## Валидация (Validation)

`@tanstack/react-form` предоставляет как синхронную, так и асинхронную валидацию "из коробки". Функции валидации можно передать в компонент `form.Field` через проп `validators`.

Пример:

```tsx
<form.Field
  name="firstName"
  validators={{
    onChange: ({ value }) =>
      !value
        ? 'Укажите имя'
        : value.length < 3
          ? 'Имя должно содержать не менее 3 символов'
          : undefined,
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && 'Имя не должно содержать "error"'
    },
  }}
  children={(field) => (
    <>
      <input
        value={field.state.value}
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.value)}
      />
      <FieldInfo field={field} />
    </>
  )}
/>
```

## Валидация со стандартными библиотеками схем (Validation with Standard Schema Libraries)

Помимо ручной валидации, также поддерживается спецификация [Standard Schema](https://github.com/standard-schema/standard-schema).

Вы можете определить схему с помощью любой библиотеки, реализующей спецификацию, и передать её в валидатор формы или поля.

Поддерживаемые библиотеки:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```tsx
import { z } from 'zod'

const userSchema = z.object({
  age: z.number().gte(13, 'Для создания аккаунта вам должно быть 13 лет'),
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

## Реактивность (Reactivity)

`@tanstack/react-form` предлагает различные способы подписки на изменения состояния формы и полей, в частности хук `useStore(form.store)` и компонент `form.Subscribe`. Эти методы позволяют оптимизировать производительность рендеринга формы, обновляя компоненты только при необходимости.

Пример:

```tsx
const firstName = useStore(form.store, (state) => state.values.firstName)
//...
<form.Subscribe
  selector={(state) => [state.canSubmit, state.isSubmitting]}
  children={([canSubmit, isSubmitting]) => (
    <button type="submit" disabled={!canSubmit}>
      {isSubmitting ? '...' : 'Отправить'}
    </button>
  )}
/>
```

Важно помнить, что хотя проп `selector` в хуке `useStore` необязателен, настоятельно рекомендуется его указывать, так как его отсутствие приведёт к лишним перерисовкам.

```tsx
// Правильное использование
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
// Неправильное использование
const store = useStore(form.store)
```

Примечание: Использование хука `useField` для реактивности не рекомендуется, так как он предназначен для осмысленного использования внутри компонента `form.Field`. Вместо него можно использовать `useStore(form.store)`.

## Слушатели (Listeners)

`@tanstack/react-form` позволяет реагировать на определённые триггеры и "слушать" их для выполнения побочных эффектов.

Пример:

```tsx
<form.Field
  name="country"
  listeners={{
    onChange: ({ value }) => {
      console.log(`Страна изменена на: ${value}, сброс провинции`)
      form.setFieldValue('province', '')
    },
  }}
/>
```

Подробнее см. в [Слушатели](./listeners.md).

## Поля-массивы (Array Fields)

Поля-массивы позволяют управлять списками значений в форме, например списком хобби. Поле-массив создаётся с помощью компонента `form.Field` с пропом `mode="array"`.

При работе с полями-массивами можно использовать методы `pushValue`, `removeValue`, `swapValues` и `moveValue` для добавления, удаления и перестановки значений в массиве.

Пример:

```tsx
<form.Field
  name="hobbies"
  mode="array"
  children={(hobbiesField) => (
    <div>
      Хобби
      <div>
        {!hobbiesField.state.value.length
          ? 'Хобби не указаны.'
          : hobbiesField.state.value.map((_, i) => (
              <div key={i}>
                <form.Field
                  name={`hobbies[${i}].name`}
                  children={(field) => {
                    return (
                      <div>
                        <label htmlFor={field.name}>Название:</label>
                        <input
                          id={field.name}
                          name={field.name}
                          value={field.state.value}
                          onBlur={field.handleBlur}
                          onChange={(e) => field.handleChange(e.target.value)}
                        />
                        <button
                          type="button"
                          onClick={() => hobbiesField.removeValue(i)}
                        >
                          X
                        </button>
                        <FieldInfo field={field} />
                      </div>
                    )
                  }}
                />
                <form.Field
                  name={`hobbies[${i}].description`}
                  children={(field) => {
                    return (
                      <div>
                        <label htmlFor={field.name}>Описание:</label>
                        <input
                          id={field.name}
                          name={field.name}
                          value={field.state.value}
                          onBlur={field.handleBlur}
                          onChange={(e) => field.handleChange(e.target.value)}
                        />
                        <FieldInfo field={field} />
                      </div>
                    )
                  }}
                />
              </div>
            ))}
      </div>
      <button
        type="button"
        onClick={() =>
          hobbiesField.pushValue({
            name: '',
            description: '',
            yearsOfExperience: 0,
          })
        }
      >
        Добавить хобби
      </button>
    </div>
  )}
/>
```

## Кнопки сброса (Reset Buttons)

При использовании `<button type="reset">` вместе с методом `form.reset()` в TanStack Form необходимо предотвратить стандартное поведение HTML-сброса, чтобы избежать неожиданного сброса элементов формы (особенно `<select>`) к их начальным HTML-значениям. Используйте `event.preventDefault()` в обработчике `onClick` кнопки, чтобы предотвратить нативный сброс формы.

Пример:

```tsx
<button
  type="reset"
  onClick={(event) => {
    event.preventDefault()
    form.reset()
  }}
>
  Сбросить
</button>
```

Альтернативно можно использовать `<button type="button">`, чтобы избежать нативного сброса HTML.

```tsx
<button
  type="button"
  onClick={() => {
    form.reset()
  }}
>
  Сбросить
</button>
```

Это основные концепции и терминология, используемые в библиотеке `@tanstack/react-form`. Понимание этих концепций поможет вам эффективнее работать с библиотекой и легко создавать сложные формы.
