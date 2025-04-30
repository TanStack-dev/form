---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T22:03:32.190Z'
id: form-validation
title: Валидация формы
---

## Валидация формы и полей

Основой функциональности TanStack Form является концепция валидации. TanStack Form делает валидацию полностью настраиваемой:

- Вы можете контролировать, когда выполнять валидацию (при изменении, вводе, потере фокуса, отправке...)
- Правила валидации можно определять на уровне поля или всей формы
- Валидация может быть синхронной или асинхронной (например, в результате вызова API)

## Когда выполняется валидация?

Это зависит от вас! Компонент `<form.Field />` принимает колбэки в качестве пропсов, такие как `onChange` или `onBlur`. Эти колбэки получают текущее значение поля и объект fieldAPI, чтобы вы могли выполнить валидацию. Если обнаружена ошибка, просто верните сообщение об ошибке в виде строки, и оно будет доступно в `field.state.meta.errors`.

Вот пример:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

В примере выше валидация выполняется при каждом нажатии клавиши (`onchange`). Если бы мы хотели выполнять валидацию при потере фокуса полем, код изменился бы так:

```svelte
<form.Field
  name="age"
  validators={{
    onBlur: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

Таким образом, вы можете контролировать момент валидации, реализуя нужный колбэк. Можно даже выполнять разные проверки в разное время:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
    onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

В примере выше мы проверяем разные условия для одного поля в разное время (при каждом нажатии клавиши и при потере фокуса). Поскольку `field.state.meta.errors` является массивом, отображаются все актуальные ошибки на данный момент. Также можно использовать `field.state.meta.errorMap` для получения ошибок в зависимости от момента валидации (onChange, onBlur и т.д.). Подробнее о выводе ошибок ниже.

## Отображение ошибок

После настройки валидации можно преобразовать ошибки из массива для отображения в интерфейсе:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

Или использовать свойство `errorMap` для доступа к конкретной ошибке:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    {#if field.state.meta.errorMap['onChange']}
      <em role="alert">{field.state.meta.errorMap['onChange']}</em>
    {/if}
  {/snippet}
</form.Field>
```

Стоит отметить, что массив `errors` и `errorMap` соответствуют типам, возвращаемым валидаторами. Это означает:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    <!-- errorMap.onChange имеет тип `{isOldEnough: false} | undefined` -->
    <!-- meta.errors имеет тип `Array<{isOldEnough: false} | undefined>` -->
    {#if field.state.meta.errorMap['onChange']?.isOldEnough}
        <em>The user is not old enough</em>
    {/if}
  {/snippet}
</form.Field>
```

## Валидация на уровне поля и формы

Как показано выше, каждый `<form.Field>` принимает собственные правила валидации через колбэки `onChange`, `onBlur` и т.д. Также можно определить правила валидации на уровне всей формы (вместо отдельных полей), передав аналогичные колбэки в хук `createForm()`.

Пример:

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      age: 0,
    },
    onSubmit: async ({ value }) => {
      console.log(value)
    },
    validators: {
      // Добавляем валидаторы для формы так же, как и для поля
      onChange({ value }) {
        if (value.age < 13) {
          return 'Must be 13 or older to sign'
        }
        return undefined
      },
    },
  }))

  // Подписываемся на карту ошибок формы для обновлений
  // альтернативно можно использовать `form.Subscribe`
  const formErrorMap = form.useStore((state) => state.errorMap)
</script>

<div>
  <!-- ... -->
  {#if formErrorMap.current.onChange}
    <div>
      <em>There was an error on the form: {formErrorMap.current.onChange}</em>
    </div>
  {/if}
  <!-- ... -->
</div>
```

## Асинхронная функциональная валидация

Хотя большинство валидаций будут синхронными, часто требуется выполнить сетевой запрос или другую асинхронную операцию для проверки.

Для этого предусмотрены специальные методы `onChangeAsync`, `onBlurAsync` и другие:

```svelte
<form.Field
  name="age"
  validators={{
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value < 13 ? 'You must be 13 to make an account' : undefined
    },
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

Синхронные и асинхронные валидации могут сосуществовать. Например, можно определить и `onBlur`, и `onBlurAsync` для одного поля:

```svelte
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
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

Сначала выполняется синхронная валидация (`onBlur`), и асинхронная (`onBlurAsync`) запускается только при успешном завершении синхронной. Чтобы изменить это поведение, установите опцию `asyncAlways` в `true`, и асинхронный метод будет выполняться независимо от результата синхронного.

### Встроенный дебаунсинг

Хотя асинхронные вызовы — правильный выбор для проверки данных в базе, выполнение сетевого запроса при каждом нажатии клавиши может привести к DDoS-атаке на вашу базу данных.

Вместо этого мы предлагаем простой способ дебаунсинга асинхронных вызовов с помощью одного свойства:

```svelte
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsync: async ({ value }) => {
      // ...
    },
  }}
>
  <!-- ... -->
</form.Field>
```

Это установит задержку в 500 мс для каждого асинхронного вызова. Можно даже переопределить это свойство для отдельных валидаций:

```svelte
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
>
  <!-- ... -->
</form.Field>
```

> В этом случае `onChangeAsync` будет выполняться каждые 1500 мс, а `onBlurAsync` — каждые 500 мс.

## Валидация с помощью библиотек схем

Хотя функции обеспечивают большую гибкость и настройку валидации, они могут быть многословными. Для решения этой проблемы существуют библиотеки, предоставляющие валидацию на основе схем, что значительно упрощает создание кратких и строго типизированных проверок. Можно определить единую схему для всей формы и передать её на уровне формы — ошибки будут автоматически распространяться на поля.

### Стандартные библиотеки схем

TanStack Form нативно поддерживает все библиотеки, соответствующие [спецификации Standard Schema](https://github.com/standard-schema/standard-schema), в частности:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_Примечание:_ убедитесь, что используете последнюю версию библиотек схем, так как старые версии могут ещё не поддерживать Standard Schema.

Чтобы использовать схемы из этих библиотек, передайте их в пропс `validators`, как и обычную функцию:

```svelte
<script>
  import { z } from 'zod'

  // ...

  const form = createForm(() => ({
    // ...
  }))
</script>

<form.Field
  name="age"
  validators={{
    onChange: z.number().gte(13, 'You must be 13 to make an account'),
  }}
>
  <!-- ... -->
</form.Field>
```

Асинхронные валидации также поддерживаются на уровне формы и полей:

```svelte
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
>
  <!-- ... -->
</form.Field>
```

## Предотвращение отправки невалидных форм

Колбэки `onChange`, `onBlur` и т.д. также выполняются при отправке формы, и отправка блокируется, если форма невалидна.

Объект состояния формы имеет флаг `canSubmit`, который становится `false`, если любое поле невалидно и форма была изменена (`canSubmit` остаётся `true`, пока форма не была изменена, даже если некоторые поля "технически" невалидны согласно их пропсам `onChange`/`onBlur`).

Можно подписаться на него через `form.Subscribe` и использовать значение для, например, отключения кнопки отправки при невалидной форме (на практике отключённые кнопки не соответствуют требованиям доступности, используйте `aria-disabled`).

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    /* ... */
  }))
</script>

<!-- ... -->

<!-- Динамическая кнопка отправки -->
<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
  children={(state) => (
    <button type="submit" disabled={!state().canSubmit}>
      {state().isSubmitting ? '...' : 'Submit'}
    </button>
  )}
>
  {#snippet children(state)}
    <button type="submit" disabled={!state.canSubmit}>
      {state.isSubmitting ? '...' : 'Submit'}
    </button>
  {/snippet}
</form.Subscribe>
```
