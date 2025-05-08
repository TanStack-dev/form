---
source-updated-at: '2025-05-08T07:42:29.000Z'
translation-updated-at: '2025-05-08T23:52:52.172Z'
id: basic-concepts
title: Основные понятия
---

# Основные концепции и терминология

На этой странице представлены основные концепции и терминология, используемые в библиотеке `@tanstack/react-form`. Ознакомление с этими понятиями поможет вам лучше понять и эффективнее работать с библиотекой.

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

Экземпляр формы (Form Instance) — это объект, представляющий отдельную форму и предоставляющий методы и свойства для работы с ней. Экземпляр формы создаётся с помощью хука `useForm`, который принимает объект с функцией `onSubmit`, вызываемой при отправке формы.

```tsx
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // Действия с данными формы
    console.log(value)
  },
})
```

Также можно создать экземпляр формы без использования `formOptions`, используя автономный API `useForm`:

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
    // Действия с данными формы
    console.log(value)
  },
})
```

## Поле (Field)

Поле (Field) представляет собой отдельный элемент ввода формы, например, текстовое поле или чекбокс. Поля создаются с помощью компонента `form.Field`, предоставляемого экземпляром формы. Компонент принимает проп `name`, который должен соответствовать ключу в значениях формы по умолчанию, а также проп `children` — функцию рендера, принимающую объект поля в качестве аргумента.

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

Каждое поле имеет собственное состояние (Field State), включающее текущее значение, статус валидации, сообщения об ошибках и другие метаданные. Доступ к состоянию поля можно получить через свойство `field.state`.

Пример:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

В метаданных есть три состояния, которые могут быть полезны для отслеживания взаимодействия пользователя с полем:

- `"isTouched"` — после того, как пользователь кликнул/перешёл в поле.
- `"isPristine"` — пока пользователь не изменил значение поля.
- `"isDirty"` — после изменения значения поля.

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Состояния поля](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## Понимание `isDirty` в разных библиотеках

**Непостоянное (Non-Persistent) состояние `dirty`**

- **Библиотеки**: React Hook Form (RHF), Formik, Final Form.
- **Поведение**: Поле считается `dirty`, если его значение отличается от значения по умолчанию. Возврат к значению по умолчанию делает его `clean`.

**Постоянное (Persistent) состояние `dirty`**

- **Библиотеки**: Angular Form, Vue FormKit.
- **Поведение**: Поле остаётся `dirty` после изменения, даже если вернуться к значению по умолчанию.

Мы выбрали модель с **постоянным `dirty`**. Для поддержки непостоянного `dirty` введён флаг `isDefault`. Этот флаг действует как инверсия непостоянного `dirty`.

```tsx
const { isTouched, isPristine, isDirty, isDefaultValue } = field.state.meta

// Следующая строка воссоздаёт функциональность непостоянного `dirty`.
const nonPersistentIsDirty = !isDefaultValue
```

![Расширенные состояния поля](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states-extended.png)

## API поля (Field API)

API поля (Field API) — это объект, передаваемый в функцию рендера при создании поля. Он предоставляет методы для работы с состоянием поля.

Пример:

```tsx
<input
  value={field.state.value}
  onBlur={field.handleBlur}
  onChange={(e) => field.handleChange(e.target.value)}
/>
```

## Валидация

`@tanstack/react-form` поддерживает синхронную и асинхронную валидацию "из коробки". Функции валидации можно передать в компонент `form.Field` через проп `validators`.

Пример:

```tsx
<form.Field
  name="firstName"
  validators={{
    onChange: ({ value }) =>
      !value
        ? 'Имя обязательно'
        : value.length < 3
          ? 'Имя должно содержать минимум 3 символа'
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

## Валидация с использованием стандартных библиотек схем (Standard Schema Libraries)

Помимо ручной валидации, поддерживается спецификация [Standard Schema](https://github.com/standard-schema/standard-schema).

Вы можете определить схему с помощью любой библиотеки, реализующей эту спецификацию, и передать её в валидатор формы или поля.

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

`@tanstack/react-form` предлагает различные способы подписки на изменения состояния формы и полей, включая хук `useStore(form.store)` и компонент `form.Subscribe`. Эти методы позволяют оптимизировать производительность рендеринга, обновляя компоненты только при необходимости.

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

Важно помнить, что хотя проп `selector` в хуке `useStore` необязателен, его использование рекомендуется, так как его отсутствие приведёт к лишним ререндерам.

```tsx
// Правильное использование
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
// Неправильное использование
const store = useStore(form.store)
```

Примечание: Использование хука `useField` для реактивности не рекомендуется, так как он предназначен для аккуратного использования внутри компонента `form.Field`. Вместо него можно использовать `useStore(form.store)`.

## Слушатели (Listeners)

`@tanstack/react-form` позволяет реагировать на определённые триггеры и "слушать" их для выполнения побочных эффектов.

Пример:

```tsx
<form.Field
  name="country"
  listeners={{
    onChange: ({ value }) => {
      console.log(`Страна изменена на: ${value}, сброс региона`)
      form.setFieldValue('province', '')
    },
  }}
/>
```

Подробнее: [Слушатели](./listeners.md)

## Поля-массивы (Array Fields)

Поля-массивы (Array Fields) позволяют управлять списками значений в форме, например, списком хобби. Такое поле создаётся с помощью компонента `form.Field` с пропом `mode="array"`.

Для работы с массивами можно использовать методы `pushValue`, `removeValue`, `swapValues` и `moveValue` для добавления, удаления и перестановки значений.

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
          ? 'Хобби не найдены.'
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

При использовании `<button type="reset">` вместе с методом `form.reset()` из TanStack Form необходимо предотвратить стандартное поведение HTML, чтобы избежать неожиданного сброса элементов формы (особенно `<select>`) к их начальным значениям.

Используйте `event.preventDefault()` в обработчике `onClick` кнопки:

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

Либо можно использовать `<button type="button">`:

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

Это основные концепции и терминология, используемые в библиотеке `@tanstack/react-form`. Их понимание поможет вам эффективнее работать с библиотекой и создавать сложные формы с лёгкостью.
