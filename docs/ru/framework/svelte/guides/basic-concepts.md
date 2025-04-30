---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T22:02:45.738Z'
id: basic-concepts
title: Основные концепции
---

# Основные концепции и терминология

На этой странице представлены основные концепции и терминология, используемые в библиотеке `@tanstack/svelte-form`. Ознакомление с этими понятиями поможет вам лучше понимать и эффективнее работать с библиотекой.

## Опции формы (Form Options)

Вы можете создать общие настройки для формы, которые можно использовать в нескольких формах, с помощью функции `formOptions`.

Пример:

```ts
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## Экземпляр формы (Form Instance)

Экземпляр формы (Form Instance) — это объект, представляющий отдельную форму и предоставляющий методы и свойства для работы с ней. Экземпляр формы создается с помощью функции `createForm`. Функция принимает объект с функцией `onSubmit`, которая вызывается при отправке формы.

```ts
const form = createForm(() => ({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // Действия с данными формы
    console.log(value)
  },
}))
```

Вы также можете создать экземпляр формы без использования `formOptions`:

```ts
const form = createForm<Person>(() => ({
  onSubmit: async ({ value }) => {
    // Действия с данными формы
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

Поле (Field) представляет отдельный элемент ввода формы, например текстовое поле или чекбокс. Поля создаются с помощью компонента `form.Field`, предоставляемого экземпляром формы. Компонент принимает проп `name`, который должен соответствовать ключу в значениях формы по умолчанию. Он также принимает проп `children` — функцию рендера (render prop), которая получает объект поля в качестве аргумента.

Пример:

```svelte
<form.Field name="firstName">
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onblur={field.handleBlur}
      oninput={(e) => field.handleChange(e.target.value)}
    />
  {/snippet}
</form.Field>
```

## Состояние поля (Field State)

Каждое поле имеет собственное состояние (Field State), включающее текущее значение, статус валидации, сообщения об ошибках и другие метаданные. Состояние поля доступно через свойство `field.state`.

Пример:

```ts
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Существует три состояния поля, которые могут быть полезны для отслеживания взаимодействия пользователя с полем. Поле считается _"touched"_ (тронутым), когда пользователь кликает или переходит в него, _"pristine"_ (нетронутым) до изменения значения и _"dirty"_ (измененным) после изменения значения. Эти состояния можно проверить через флаги `isTouched`, `isPristine` и `isDirty`, как показано ниже.

```ts
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Состояния поля](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## API поля (Field API)

API поля (Field API) — это объект, передаваемый в функцию рендера при создании поля. Он предоставляет методы для работы с состоянием поля.

Пример:

```svelte
<input
  name={field.name}
  value={field.state.value}
  onblur={field.handleBlur}
  oninput={(e) => field.handleChange(e.target.value)}
/>
```

## Валидация (Validation)

`@tanstack/svelte-form` предоставляет как синхронную, так и асинхронную валидацию "из коробки". Функции валидации можно передать в компонент `form.Field` через проп `validators`.

Пример:

```svelte
<form.Field
  name="firstName"
  validators={{
    onChange: ({ value }) =>
      !value
        ? 'Имя обязательно для заполнения'
        : value.length < 3
          ? 'Имя должно содержать минимум 3 символа'
          : undefined,
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && 'Слово "error" запрещено в имени'
    },
  }}
>
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onBlur={field.handleBlur}
      onInput={(e) => field.handleChange(e.target.value)}
    />
    <p>{field.state.meta.errors[0]}</p>
  {/snippet}
</form.Field>
```

## Валидация с помощью стандартных библиотек схем (Validation with Standard Schema Libraries)

Помимо ручной настройки валидации, библиотека также поддерживает спецификацию [Standard Schema](https://github.com/standard-schema/standard-schema).

Вы можете определить схему с помощью любой библиотеки, реализующей эту спецификацию, и передать её в валидатор формы или поля.

Поддерживаемые библиотеки:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```svelte
<script>
  import { z } from 'zod'

  // ...
</script>

<form.Field
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
>
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onBlur={field.handleBlur}
      onInput={(e) => field.handleChange(e.target.value)}
    />
    <p>{field.state.meta.errors[0]}</p>
  {/snippet}
</form.Field>
```

## Реактивность (Reactivity)

`@tanstack/svelte-form` предлагает различные способы подписки на изменения состояния формы и полей, в частности хук `form.useStore` и компонент `form.Subscribe`. Эти методы позволяют оптимизировать производительность рендеринга формы, обновляя компоненты только при необходимости.

Пример:

```svelte
<script>
  //...
  const firstName = form.useStore((state) => state.values.firstName)
</script>

<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
>
  {#snippet children(state)}
    <button type="submit" disabled={!state.canSubmit}>
      {state.isSubmitting ? '...' : 'Отправить'}
    </button>
  {/snippet}
</form.Subscribe>
```

## Массивы полей (Array Fields)

Массивы полей (Array Fields) позволяют управлять списками значений в форме, например списком увлечений. Вы можете создать массив полей с помощью компонента `form.Field` с пропом `mode="array"`.

При работе с массивами полей можно использовать методы `pushValue`, `removeValue`, `swapValues` и `moveValue` для добавления, удаления и перестановки значений в массиве.

Пример:

```svelte
<form.Field name="hobbies" mode="array">
  {#snippet children(hobbiesField)}
    <div>
      Увлечения
      <div>
        {#each hobbiesField.state.value as _, i}
            <div>
              <form.Field name={`hobbies[${i}].name`}>
                {#snippet children(field)}
                  <div>
                    <label for={field.name}>Название:</label>
                    <input
                      id={field.name}
                      name={field.name}
                      value={field.state.value}
                      onblur={field.handleBlur}
                      onchange={(e) => field.handleChange(e.target.value)}
                    />
                    <button
                      type="button"
                      onclick={() => hobbiesField.removeValue(i)}
                    >
                      X
                    </button>
                  </div>
                {/snippet}
              </form.Field>
              <form.Field name={`hobbies[${i}].description`}>
                {#snippet children(field)}
                    <div>
                      <label for={field.name}>Описание:</label>
                      <input
                        id={field.name}
                        name={field.name}
                        value={field.state.value}
                        onblur={field.handleBlur}
                        onchange={(e) => field.handleChange(e.target.value)}
                      />
                    </div>
                {/snippet}
              </form.Field>
            </div>
          {:else}
            Увлечения не добавлены.
          {/each}
      </div>
      <button
        type="button"
        onclick={() =>
          hobbiesField.pushValue({
            name: '',
            description: '',
            yearsOfExperience: 0,
          })
        }
      >
        Добавить увлечение
      </button>
    </div>
  {/snippet}
</form.Field>
```

Это основные концепции и терминология, используемые в библиотеке `@tanstack/svelte-form`. Понимание этих концепций поможет вам эффективнее работать с библиотекой и создавать сложные формы с легкостью.
