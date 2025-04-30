---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T22:02:11.937Z'
id: form-validation
title: Валидация формы
---

Основой функциональности TanStack Form является концепция валидации. TanStack Form делает валидацию максимально настраиваемой:

- Вы можете контролировать, когда выполнять валидацию (при изменении, вводе, потере фокуса, отправке и т. д.)
- Правила валидации можно определять на уровне поля или на уровне формы
- Валидация может быть синхронной или асинхронной (например, в результате вызова API)

## Когда выполняется валидация?

Это зависит от вас! Компонент `<Field />` принимает колбэки в качестве пропсов, такие как `onChange` или `onBlur`. Эти колбэки получают текущее значение поля и объект `fieldAPI`, чтобы вы могли выполнить валидацию. Если обнаружена ошибка валидации, просто верните сообщение об ошибке в виде строки, и оно будет доступно в `field.state.meta.errors`.

Пример:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

В примере выше валидация выполняется при каждом нажатии клавиши (`onChange`). Если бы мы хотели выполнять валидацию при потере фокуса, код изменился бы так:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onBlur: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <!-- onChange всегда должен быть реализован, чтобы TanStack Form получал изменения -->
      <!-- Слушаем событие onBlur на поле -->
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

Таким образом, вы можете контролировать момент валидации, реализуя нужный колбэк. Можно даже выполнять разные проверки в разное время:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
      onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <!-- onChange всегда должен быть реализован, чтобы TanStack Form получал изменения -->
      <!-- Слушаем событие onBlur на поле -->
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

В примере выше мы проверяем разные условия для одного поля в разное время (при каждом нажатии клавиши и при потере фокуса). Поскольку `field.state.meta.errors` — это массив, все актуальные ошибки отображаются. Также можно использовать `field.state.meta.errorMap` для получения ошибок в зависимости от момента валидации (onChange, onBlur и т. д.). Подробнее о выводе ошибок ниже.

## Отображение ошибок

После настройки валидации можно преобразовать ошибки из массива для отображения в интерфейсе:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

Или использовать свойство `errorMap` для доступа к конкретной ошибке:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
      <em role="alert" v-if="field.state.meta.errorMap['onChange']">{{
        field.state.meta.errorMap['onChange']
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

Важно отметить, что массив `errors` и `errorMap` соответствуют типам, возвращаемым валидаторами. Это означает:

```vue
<form.Field
  name="age"
  :validators="{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }"
>
  <template v-slot="{ field }">
      <!-- ... -->
      <!-- errorMap.onChange имеет тип `{isOldEnough: false} | undefined` -->
	  <!-- meta.errors имеет тип `Array<{isOldEnough: false} | undefined>` -->
      <em v-if="!field.state.meta.errorMap['onChange']?.isOldEnough">The user is not old enough</em>
  </template>
</form.Field>
```

## Валидация на уровне поля и формы

Как показано выше, каждый `<Field>` принимает свои правила валидации через колбэки `onChange`, `onBlur` и т. д. Также можно определить правила валидации на уровне формы (а не для каждого поля отдельно), передав аналогичные колбэки в функцию `useForm()`.

Пример:

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

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

// Подписываемся на errorMap формы для обновлений
// Альтернативно можно использовать `form.Subscribe`
const formErrorMap = form.useStore((state) => state.errorMap)
</script>

<template>
  <!-- ... -->
  <div v-if="formErrorMap.onChange">
    <em role="alert">
      There was an error on the form: {{ formErrorMap.onChange }}
    </em>
  </div>
  <!-- ... -->
</template>
```

### Установка ошибок на уровне поля из валидаторов формы

Можно устанавливать ошибки для полей из валидаторов формы. Один из распространённых случаев — валидация всех полей при отправке формы через один API-запрос в валидаторе `onSubmitAsync`.

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    age: 0,
    socials: [],
    details: {
      email: '',
    },
  },
  validators: {
    // Добавляем валидаторы к форме так же, как к полю
    onSubmitAsync({ value }) {
      // Валидация значения на сервере
      const hasErrors = await verifyDataOnServer(value)
      if (hasErrors) {
        return {
          form: 'Invalid data', // Ключ `form` не обязателен
          fields: {
            age: 'Must be 13 or older to sign',
            // Установка ошибок для вложенных полей по имени
            'socials[0].url': 'The provided URL does not exist',
            'details.email': 'An email is required',
          },
        }
      }

      return null
    },
  },
})
</script>

<template>
  <!-- ... -->
</template>
```

> Важно: если функция валидации формы возвращает ошибку, она может быть перезаписана валидацией конкретного поля.
>
> Это означает:
>
> ```vue
> <script setup lang="ts">
> import { useForm } from '@tanstack/vue-form'
>
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
> </script>
>
> <template>
>   <!-- ... -->
>   <form.Field
>     name="age"
>     :validators="{
>       onChange: ({ value }) => (value % 2 === 0 ? 'Must be odd!' : undefined),
>     }"
>   >
>     <template v-slot="{ field }">
>       <!-- ... -->
>     </template>
>   </form.Field>
> </template>
> ```
>
> Будет отображаться только `'Must be odd!'`, даже если валидация формы вернула ошибку 'Too young!'.

## Асинхронная функциональная валидация

Хотя большинство валидаций, вероятно, будут синхронными, часто требуется выполнить сетевой запрос или другую асинхронную операцию для проверки.

Для этого предусмотрены специальные методы `onChangeAsync`, `onBlurAsync` и другие:

```vue
<script setup lang="ts">
// ...

const onChangeAge = async ({ value }) => {
  await new Promise((resolve) => setTimeout(resolve, 1000))
  return value < 13 ? 'You must be 13 to make an account' : undefined
}
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChangeAsync: onChangeAge,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @input="
          (e) =>
            field.handleChange((e.target as HTMLInputElement).valueAsNumber)
        "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

Синхронная и асинхронная валидации могут сосуществовать. Например, можно определить и `onBlur`, и `onBlurAsync` для одного поля:

```vue
<script setup lang="ts">
// ...

const onBlurAge = ({ value }) => (value < 0 ? 'Invalid value' : undefined)

const onBlurAgeAsync = async ({ value }) => {
  const currentAge = await fetchCurrentAgeOnProfile()
  return value < currentAge ? 'You can only increase the age' : undefined
}
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onBlur: onBlurAge,
      onBlurAsync: onBlurAgeAsync,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="
          (e) =>
            field.handleChange((e.target as HTMLInputElement).valueAsNumber)
        "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

Сначала выполняется синхронная валидация (`onBlur`), а асинхронная (`onBlurAsync`) — только если синхронная прошла успешно. Чтобы изменить это поведение, установите параметр `asyncAlways` в `true`, и асинхронный метод будет выполняться независимо от результата синхронного.

### Встроенный дебаунсинг

Хотя асинхронные вызовы — это правильный подход для валидации данных в базе, выполнение сетевого запроса при каждом нажатии клавиши может привести к DDoS-атаке на вашу базу данных.

Вместо этого мы предлагаем простой способ дебаунсинга асинхронных вызовов с помощью одного свойства:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :async-debounce-ms="500"
    :validators="{
      onChangeAsync: async ({ value }) => {
        // ...
      },
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

Это установит задержку в 500 мс для всех асинхронных вызовов. Можно даже переопределить это свойство для отдельных валидаций:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :async-debounce-ms="500"
    :validators="{
      onChangeAsyncDebounceMs: 1500,
      onChangeAsync: async ({ value }) => {
        // ...
      },
      onBlurAsync: async ({ value }) => {
        // ...
      },
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

В этом случае `onChangeAsync` будет выполняться каждые 1500 мс, а `onBlurAsync` — каждые 500 мс.

## Валидация с помощью библиотек схем

Хотя функции предоставляют больше гибкости и возможностей для валидации, они могут быть многословными. Для упрощения существуют библиотеки, предлагающие валидацию на основе схем, что делает краткую и строго типизированную валидацию значительно проще. Можно также определить одну схему для всей формы и передать её на уровне формы — ошибки будут автоматически распространяться на поля.

### Стандартные библиотеки схем

TanStack Form нативно поддерживает все библиотеки, соответствующие [спецификации Standard Schema](https://github.com/standard-schema/standard-schema), включая:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_Примечание:_ убедитесь, что используете последнюю версию библиотек схем, так как старые версии могут ещё не поддерживать Standard Schema.

> Валидация не предоставляет преобразованных значений. Подробнее см. в разделе [обработка отправки](./submission-handling.md).

Чтобы использовать схемы из этих библиотек, передайте их в проп `validators` как обычную функцию:

```vue
<script setup lang="ts">
import { z } from 'zod'
// ...

const form = useForm({
  // ...
})
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: z.number().gte(13, 'You must be 13 to make an account'),
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

Асинхронная валидация на уровне формы и полей также поддерживается:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
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
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

Если требуется больше контроля над валидацией Standard Schema, можно комбинировать схему с функцией-колбэком:

```

```
