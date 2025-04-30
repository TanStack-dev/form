---
source-updated-at: '2025-03-14T21:54:05.000Z'
translation-updated-at: '2025-04-30T22:01:42.463Z'
id: basic-concepts
title: Основные концепции
---

# Основные концепции и терминология

На этой странице представлены основные концепции и терминология, используемые в библиотеке `@tanstack/solid-form`. Ознакомление с этими понятиями поможет вам лучше понять и работать с библиотекой.

## Настройки формы (Form Options)

Вы можете создать настройки для своей формы, чтобы их можно было использовать в нескольких формах, с помощью функции `formOptions`.

Пример:

```tsx
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## Экземпляр формы (Form Instance)

Экземпляр формы (Form Instance) — это объект, представляющий отдельную форму и предоставляющий методы и свойства для работы с ней. Вы создаёте экземпляр формы с помощью хука `createForm`, предоставляемого настройками формы. Хук принимает объект с функцией `onSubmit`, которая вызывается при отправке формы.

```tsx
const form = createForm(() => ({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // Делаем что-то с данными формы
    console.log(value)
  },
}))
```

Вы также можете создать экземпляр формы без использования `formOptions`, воспользовавшись автономным API `createForm`:

```tsx
const form = createForm<Person>(() => ({
  onSubmit: async ({ value }) => {
    // Делаем что-то с данными формы
    console.log(value)
  },
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  },
}))
```

## Поле (Field)

Поле (Field) представляет отдельный элемент ввода формы, например текстовое поле или чекбокс. Поля создаются с помощью компонента `form.Field`, предоставляемого экземпляром формы. Компонент принимает проп `name`, который должен соответствовать ключу в значениях формы по умолчанию. Он также принимает проп `children`, который является функцией рендеринга, получающей объект поля в качестве аргумента.

Пример:

```tsx
<form.Field
  name="firstName"
  children={(field) => (
    <input
      name={field().name}
      value={field().state.value}
      onBlur={field().handleBlur}
      onInput={(e) => field().handleChange(e.target.value)}
    />
  )}
/>
```

## Состояние поля (Field State)

Каждое поле имеет собственное состояние, включающее текущее значение, статус валидации, сообщения об ошибках и другие метаданные. Вы можете получить доступ к состоянию поля с помощью свойства `field().state`.

Пример:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field().state
```

Существует три состояния поля, которые могут быть полезны для отслеживания взаимодействия пользователя с полем. Поле считается _"touched"_ (тронутым), когда пользователь кликает/переходит в него, _"pristine"_ (нетронутым) до изменения значения и _"dirty"_ (изменённым) после изменения значения. Эти состояния можно проверить с помощью флагов `isTouched`, `isPristine` и `isDirty`, как показано ниже.

```tsx
const { isTouched, isPristine, isDirty } = field().state.meta
```

![Состояния поля](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## API поля (Field API)

API поля (Field API) — это объект, передаваемый в функцию рендеринга при создании поля. Он предоставляет методы для работы с состоянием поля.

Пример:

```tsx
<input
  name={field().name}
  value={field().state.value}
  onBlur={field().handleBlur}
  onInput={(e) => field().handleChange(e.target.value)}
/>
```

## Валидация (Validation)

`@tanstack/solid-form` предоставляет как синхронную, так и асинхронную валидацию "из коробки". Функции валидации можно передать в компонент `form.Field` с помощью пропа `validators`.

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
      return value.includes('error') && 'Слово "error" запрещено в имени'
    },
  }}
  children={(field) => (
    <>
      <input
        name={field().name}
        value={field().state.value}
        onBlur={field().handleBlur}
        onInput={(e) => field().handleChange(e.target.value)}
      />
      <p>{field().state.meta.errors[0]}</p>
    </>
  )}
/>
```

## Валидация с помощью стандартных библиотек схем (Validation with Standard Schema Libraries)

В дополнение к ручной валидации, мы также поддерживаем спецификацию [Standard Schema](https://github.com/standard-schema/standard-schema).

Вы можете определить схему с помощью любой из библиотек, реализующих спецификацию, и передать её валидатору формы или поля.

Поддерживаемые библиотеки:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```tsx
import { z } from 'zod'

// ...
;<form.Field
  name="firstName"
  validators={{
    onChange: z.string().min(3, 'Имя должно содержать минимум 3 символа'),
    onChangeAsyncDebounceMs: 500,
    onChangeAsync: z.string().refine(
      async (value) => {
        await new Promise((resolve) => setTimeout(resolve, 1000))
        return !value.includes('error')
      },
      {
        message: 'Слово "error" запрещено в имени',
      },
    ),
  }}
  children={(field) => (
    <>
      <input
        name={field().name}
        value={field().state.value}
        onBlur={field().handleBlur}
        onInput={(e) => field().handleChange(e.target.value)}
      />
      <p>{field().state.meta.errors[0]}</p>
    </>
  )}
/>
```

## Реактивность (Reactivity)

`@tanstack/solid-form` предлагает различные способы подписки на изменения состояния формы и полей, в частности хук `form.useStore` и компонент `form.Subscribe`. Эти методы позволяют оптимизировать производительность рендеринга формы, обновляя компоненты только при необходимости.

Пример:

```tsx
const firstName = form.useStore((state) => state.values.firstName)
//...
<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
  children={(state) => (
    <button type="submit" disabled={!state().canSubmit}>
      {state().isSubmitting ? '...' : 'Отправить'}
    </button>
  )}
/>
```

## Поля-массивы (Array Fields)

Поля-массивы (Array Fields) позволяют управлять списком значений в форме, например списком хобби. Вы можете создать поле-массив с помощью компонента `form.Field` с пропом `mode="array"`.

При работе с полями-массивами вы можете использовать методы `pushValue`, `removeValue`, `swapValues` и `moveValue` для добавления, удаления и перестановки значений в массиве.

Пример:

```tsx
<form.Field
  name="hobbies"
  mode="array"
  children={(hobbiesField) => (
    <div>
      Хобби
      <div>
        <Show
          when={hobbiesField().state.value.length > 0}
          fallback={'Хобби не найдены.'}
        >
          <Index each={hobbiesField().state.value}>
            {(_, i) => (
              <div>
                <form.Field
                  name={`hobbies[${i}].name`}
                  children={(field) => (
                    <div>
                      <label for={field().name}>Название:</label>
                      <input
                        id={field().name}
                        name={field().name}
                        value={field().state.value}
                        onBlur={field().handleBlur}
                        onInput={(e) => field().handleChange(e.target.value)}
                      />
                      <button
                        type="button"
                        onClick={() => hobbiesField().removeValue(i)}
                      >
                        X
                      </button>
                    </div>
                  )}
                />
                <form.Field
                  name={`hobbies[${i}].description`}
                  children={(field) => {
                    return (
                      <div>
                        <label for={field().name}>Описание:</label>
                        <input
                          id={field().name}
                          name={field().name}
                          value={field().state.value}
                          onBlur={field().handleBlur}
                          onInput={(e) => field().handleChange(e.target.value)}
                        />
                      </div>
                    )
                  }}
                />
              </div>
            )}
          </Index>
        </Show>
      </div>
      <button
        type="button"
        onClick={() =>
          hobbiesField().pushValue({
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

Это основные концепции и терминология, используемые в библиотеке `@tanstack/solid-form`. Понимание этих концепций поможет вам эффективнее работать с библиотекой и легко создавать сложные формы.
