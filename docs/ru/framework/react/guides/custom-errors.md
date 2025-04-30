---
source-updated-at: '2025-03-04T14:23:57.000Z'
translation-updated-at: '2025-04-30T22:00:20.606Z'
id: custom-errors
title: Пользовательские ошибки
---

TanStack Form предоставляет полную гибкость в типах ошибок, которые можно возвращать из валидаторов. Строковые ошибки являются наиболее распространёнными и удобными для работы, но библиотека позволяет возвращать из валидаторов значения любого типа.

Общее правило: любое истинное значение (truthy) считается ошибкой и помечает форму или поле как невалидное, тогда как ложные значения (`false`, `undefined`, `null` и т.д.) означают отсутствие ошибки, и форма или поле считаются валидными.

## Возврат строковых значений из форм

```tsx
<form.Field
  name="username"
  validators={{
    onChange: ({ value }) =>
      value.length < 3 ? 'Username must be at least 3 characters' : undefined,
  }}
/>
```

Для валидации на уровне формы, затрагивающей несколько полей:

```tsx
const form = useForm({
  defaultValues: {
    username: '',
    email: '',
  },
  validators: {
    onChange: ({ value }) => {
      return {
        fields: {
          username:
            value.username.length < 3 ? 'Username too short' : undefined,
          email: !value.email.includes('@') ? 'Invalid email' : undefined,
        },
      }
    },
  },
})
```

Строковые ошибки — наиболее распространённый тип, их легко отображать в интерфейсе:

```tsx
{
  field.state.meta.errors.map((error, i) => (
    <div key={i} className="error">
      {error}
    </div>
  ))
}
```

### Числа

Полезны для представления количеств, пороговых значений или величин:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 18 ? 18 - value : undefined),
  }}
/>
```

Отображение в интерфейсе:

```tsx
{
  /* TypeScript знает, что ошибка — это число, на основе вашего валидатора */
}
;<div className="error">
  Вам нужно ещё {field.state.meta.errors[0]} лет для соответствия требованиям
</div>
```

### Логические значения

Простые флаги для указания состояния ошибки:

```tsx
<form.Field
  name="accepted"
  validators={{
    onChange: ({ value }) => (!value ? true : undefined),
  }}
/>
```

Отображение в интерфейсе:

```tsx
{
  field.state.meta.errors[0] === true && (
    <div className="error">Необходимо принять условия</div>
  )
}
```

### Объекты

Сложные объекты ошибок с несколькими свойствами:

```tsx
<form.Field
  name="email"
  validators={{
    onChange: ({ value }) => {
      if (!value.includes('@')) {
        return {
          message: 'Неверный формат email',
          severity: 'error',
          code: 1001,
        }
      }
      return undefined
    },
  }}
/>
```

Отображение в интерфейсе:

```tsx
{
  typeof field.state.meta.errors[0] === 'object' && (
    <div className={`error ${field.state.meta.errors[0].severity}`}>
      {field.state.meta.errors[0].message}
      <small> (Код: {field.state.meta.errors[0].code})</small>
    </div>
  )
}
```

В примере выше это зависит от типа ошибки, которую вы хотите отобразить.

### Массивы

Несколько сообщений об ошибках для одного поля:

```tsx
<form.Field
  name="password"
  validators={{
    onChange: ({ value }) => {
      const errors = []
      if (value.length < 8) errors.push('Пароль слишком короткий')
      if (!/[A-Z]/.test(value)) errors.push('Отсутствует заглавная буква')
      if (!/[0-9]/.test(value)) errors.push('Отсутствует цифра')

      return errors.length ? errors : undefined
    },
  }}
/>
```

Отображение в интерфейсе:

```tsx
{
  Array.isArray(field.state.meta.errors) && (
    <ul className="error-list">
      {field.state.meta.errors.map((err, i) => (
        <li key={i}>{err}</li>
      ))}
    </ul>
  )
}
```

## Свойство `disableErrorFlat` для полей

По умолчанию TanStack Form объединяет ошибки из всех источников валидации (onChange, onBlur, onSubmit) в единый массив `errors`. Свойство `disableErrorFlat` сохраняет разделение ошибок по их источникам:

```tsx
<form.Field
  name="email"
  disableErrorFlat
  validators={{
    onChange: ({ value }) =>
      !value.includes('@') ? 'Неверный формат email' : undefined,
    onBlur: ({ value }) =>
      !value.endsWith('.com') ? 'Разрешены только домены .com' : undefined,
    onSubmit: ({ value }) =>
      value.length < 5 ? 'Email слишком короткий' : undefined,
  }}
/>
```

Без `disableErrorFlat` все ошибки объединяются в `field.state.meta.errors`. С этим свойством можно обращаться к ошибкам по их источнику:

```tsx
{
  field.state.meta.errorMap.onChange && (
    <div className="real-time-error">{field.state.meta.errorMap.onChange}</div>
  )
}

{
  field.state.meta.errorMap.onBlur && (
    <div className="blur-feedback">{field.state.meta.errorMap.onBlur}</div>
  )
}

{
  field.state.meta.errorMap.onSubmit && (
    <div className="submit-error">{field.state.meta.errorMap.onSubmit}</div>
  )
}
```

Это полезно для:

- Различного оформления ошибок разных типов
- Приоритизации ошибок (например, более заметного отображения ошибок отправки)
- Постепенного раскрытия ошибок

## Типобезопасность `errors` и `errorMap`

TanStack Form обеспечивает строгую типобезопасность при обработке ошибок. Каждый ключ в `errorMap` имеет в точности тип, возвращаемый соответствующим валидатором, тогда как массив `errors` содержит объединённый тип всех возможных значений ошибок из всех валидаторов:

```tsx
<form.Field
  name="password"
  validators={{
    onChange: ({ value }) => {
      // Возвращает строку или undefined
      return value.length < 8 ? 'Слишком короткий' : undefined
    },
    onBlur: ({ value }) => {
      // Возвращает объект или undefined
      if (!/[A-Z]/.test(value)) {
        return { message: 'Отсутствует заглавная буква', level: 'warning' }
      }
      return undefined
    },
  }}
  children={(field) => {
    // TypeScript знает, что errors[0] может быть string | { message: string, level: string } | undefined
    const error = field.state.meta.errors[0]

    // Типобезопасная обработка ошибок
    if (typeof error === 'string') {
      return <div className="string-error">{error}</div>
    } else if (error && typeof error === 'object') {
      return <div className={error.level}>{error.message}</div>
    }

    return null
  }}
/>
```

Свойство `errorMap` также полностью типизировано и соответствует типам возвращаемых значений ваших валидационных функций:

```tsx
// С disableErrorFlat
<form.Field
  name="email"
  disableErrorFlat
  validators={{
    onChange: ({ value }): string | undefined =>
      !value.includes("@") ? "Неверный email" : undefined,
    onBlur: ({ value }): { code: number, message: string } | undefined =>
      !value.endsWith(".com") ? { code: 100, message: "Неправильный домен" } : undefined
  }}
  children={(field) => {
    // TypeScript знает точный тип каждого источника ошибки
    const onChangeError: string | undefined = field.state.meta.errorMap.onChange;
    const onBlurError: { code: number, message: string } | undefined = field.state.meta.errorMap.onBlur;

    return (/* ... */);
  }}
/>
```

Эта типобезопасность помогает выявлять ошибки на этапе компиляции, а не во время выполнения, делая ваш код более надёжным и поддерживаемым.
