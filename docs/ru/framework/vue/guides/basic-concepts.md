---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T22:01:00.552Z'
id: basic-concepts
title: Основные концепции
---

# Основные концепции и терминология

На этой странице представлены основные концепции и терминология, используемые в библиотеке `@tanstack/vue-form`. Ознакомление с этими понятиями поможет вам лучше понимать и эффективнее работать с библиотекой.

## Настройки формы (Form Options)

Вы можете создать настройки для формы, чтобы использовать их в нескольких формах, с помощью функции `formOptions`.

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

Экземпляр формы — это объект, представляющий отдельную форму и предоставляющий методы и свойства для работы с ней. Экземпляр формы создаётся с помощью функции `useForm`. Функция принимает объект с функцией `onSubmit`, которая вызывается при отправке формы.

```js
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // Действия с данными формы
    console.log(value)
  },
})
```

Вы также можете создать экземпляр формы без использования `formOptions`, используя автономный API `useForm`:

```ts
const form = useForm({
  onSubmit: async ({ value }) => {
    // Действия с данными формы
    console.log(value)
  },
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## Поле (Field)

Поле представляет собой отдельный элемент ввода формы, например, текстовое поле или чекбокс. Поля создаются с помощью компонента `form.Field`, предоставляемого экземпляром формы. Компонент принимает проп `name`, который должен соответствовать ключу в значениях по умолчанию формы. Также он принимает слот с областью видимости, определяемый директивой `v-slot`, который получает объект `field` в качестве аргумента.

Пример:

```vue
<template>
  <!-- ... -->
  <form.Field name="fullName">
    <template v-slot="{ field }">
      <input
        :name="field.name"
        :value="field.state.value"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange(e.target.value)"
      />
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## Состояние поля (Field State)

Каждое поле имеет собственное состояние, включающее текущее значение, статус валидации, сообщения об ошибках и другие метаданные. Состояние поля можно получить через свойство `field.state`.

Пример:

```js
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Существует три состояния поля, которые могут быть полезны для отслеживания взаимодействия пользователя с полем. Поле считается _"touched"_ (тронутым), когда пользователь кликает или переходит в него, _"pristine"_ (нетронутым), пока пользователь не изменит его значение, и _"dirty"_ (изменённым), после изменения значения. Эти состояния можно проверить через флаги `isTouched`, `isPristine` и `isDirty`, как показано ниже.

```js
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Состояния поля](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## API поля (Field API)

API поля — это объект, предоставляемый слотом с областью видимости через директиву `v-slot`. Этот слот получает аргумент `field`, который содержит методы и свойства для работы с состоянием поля.

Пример:

```vue
<template v-slot="{ field }">
  <input
    :name="field.name"
    :value="field.state.value"
    @blur="field.handleBlur"
    @input="(e) => field.handleChange(e.target.value)"
  />
</template>
```

## Валидация (Validation)

`@tanstack/vue-form` предоставляет синхронную и асинхронную валидацию "из коробки". Функции валидации можно передать в компонент `form.Field` через проп `validators`.

Пример:

```vue
<template>
    <!-- ... -->
    <form.Field
        name="firstName"
        :validators="{
            onChange: ({ value }) =>
                !value
                    ? `Укажите имя`
                    : value.length < 3
                        ? `Имя должно содержать минимум 3 символа`
                        : undefined,
            onChangeAsync: async ({ value }) => {
                await new Promise((resolve) => setTimeout(resolve, 1000))
                return value.includes('error') && 'Запрещено использовать "error" в имени'
        },
    }"
    >
        <template v-slot="{ field }">
            <input
                :value="field.state.value"
                @input="(e) => field.handleChange(e.target.value)"
                @blur="field.handleBlur"
            />
            <FieldInfo :field="field" />
        </template>
    </form.Field>
    <!-- ... -->
</template>
```

## Валидация с помощью стандартных библиотек схем (Validation with Standard Schema Libraries)

Помимо ручной настройки валидации, библиотека также поддерживает спецификацию [Standard Schema](https://github.com/standard-schema/standard-schema).

Вы можете определить схему с помощью любой из библиотек, реализующих эту спецификацию, и передать её в валидатор формы или поля.

Поддерживаемые библиотеки:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'
import { z } from 'zod'

const form = useForm({
  // ...
})

const onChangeFirstName = z.string().refine(
  async (value) => {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return !value.includes('error')
  },
  {
    message: "Запрещено использовать 'error' в имени",
  },
)
</script>

<template>
  <!-- ... -->
  <form.Field
    name="firstName"
    :validators="{
      onChange: z.string().min(3, 'Имя должно содержать минимум 3 символа'),
      onChangeAsyncDebounceMs: 500,
      onChangeAsync: onChangeFirstName,
    }"
  >
    <template v-slot="{ field, state }">
      <label :htmlFor="field.name">Имя:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).value)"
        @blur="field.handleBlur"
      />
      <FieldInfo :state="state" />
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## Реактивность (Reactivity)

`@tanstack/vue-form` предлагает различные способы подписки на изменения состояния формы и полей, включая метод `form.useStore` и компонент `form.Subscribe`. Эти методы позволяют оптимизировать производительность рендеринга формы, обновляя компоненты только при необходимости.

Пример:

```vue
<script setup lang="ts">
// ...
const firstName = form.useStore((state) => state.values.firstName)
</script>

<template>
  <!-- ... -->
  <form.Subscribe>
    <template v-slot="{ canSubmit, isSubmitting }">
      <button type="submit" :disabled="!canSubmit">
        {{ isSubmitting ? '...' : 'Отправить' }}
      </button>
    </template>
  </form.Subscribe>
  <!-- ... -->
</template>
```

## Слушатели (Listeners)

`@tanstack/vue-form` позволяет реагировать на определённые триггеры и "слушать" их для выполнения побочных эффектов.

Пример:

```vue
<template>
  <form.Field
    name="country"
    :listeners="{
      onChange: ({ value }) => {
        console.log(`Страна изменена на: ${value}, сброс региона`)
        form.setFieldValue('province', '')
      },
    }"
  >
    <template v-slot="{ field }">
      <input
        :value="field.state.value"
        @input="(e) => field.handleChange(e.target.value)"
      />
    </template>
  </form.Field>
</template>
```

Подробнее можно узнать в разделе [Слушатели](./listeners.md).

Примечание: Использование метода `form.useField` для реактивности не рекомендуется, так как он предназначен для осмысленного использования внутри компонента `form.Field`. Вместо него можно использовать `form.useStore`.

## Массивы полей (Array Fields)

Массивы полей позволяют управлять списками значений в форме, например, списком хобби. Вы можете создать массив полей с помощью компонента `form.Field` и пропа `mode="array"`.

При работе с массивами полей можно использовать методы `pushValue`, `removeValue`, `swapValues` и `moveValue` для добавления, удаления и перестановки значений в массиве.

Пример:

```vue
<template>
  <form @submit.prevent.stop="form.handleSubmit">
    <form.Field name="hobbies" mode="array">
      <template v-slot="{ field: hobbiesField }">
        <div>
          Хобби
          <div>
            <div
              v-if="
                Array.isArray(hobbiesField.state.value) &&
                !hobbiesField.state.value.length
              "
            >
              Хобби не указаны.
            </div>
            <div v-else>
              <div v-for="(_, i) in hobbiesField.state.value" :key="i">
                <form.Field :name="`hobbies[${i}].name`">
                  <template v-slot="{ field }">
                    <div>
                      <label :for="field.name">Название:</label>
                      <input
                        :id="field.name"
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="(e) => field.handleChange(e.target.value)"
                      />
                      <button
                        type="button"
                        @click="hobbiesField.removeValue(i)"
                      >
                        X
                      </button>
                      <FieldInfo :field="field" />
                    </div>
                  </template>
                </form.Field>
                <form.Field :name="`hobbies[${i}].description`">
                  <template v-slot="{ field }">
                    <div>
                      <label :for="field.name">Описание:</label>
                      <input
                        :id="field.name"
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="(e) => field.handleChange(e.target.value)"
                      />
                      <FieldInfo :field="field" />
                    </div>
                  </template>
                </form.Field>
              </div>
            </div>
            <button
              type="button"
              @click="
                hobbiesField.pushValue({
                  name: '',
                  description: '',
                  yearsOfExperience: 0,
                })
              "
            >
              Добавить хобби
            </button>
          </div>
        </div>
      </template>
    </form.Field>
  </form>
</template>
```

Это основные концепции и терминология, используемые в библиотеке `@tanstack/vue-form`. Понимание этих понятий поможет вам эффективнее работать с библиотекой и создавать сложные формы с лёгкостью.
