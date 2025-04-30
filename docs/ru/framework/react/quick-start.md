---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T21:59:12.171Z'
id: quick-start
title: Быстрый старт
---

# Быстрый старт

TanStack Form отличается от большинства библиотек для работы с формами, которые вы использовали ранее. Он разработан для масштабируемого промышленного применения с акцентом на безопасность типов (type safety), производительность и композицию, чтобы обеспечить непревзойденный опыт разработчика.

В результате мы разработали [философию использования библиотеки](/form/latest/docs/philosophy), которая ставит во главу угла масштабируемость и долгосрочный опыт разработчика, а не короткие и легко копируемые фрагменты кода.

Вот пример формы, следующей многим нашим лучшим практикам, который позволит вам быстро разрабатывать даже формы высокой сложности после короткого периода адаптации:

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { createFormHook, createFormHookContexts } from '@tanstack/react-form'
// Компоненты формы, которые предварительно связывают события с хуком формы; подробнее см. в руководстве "Form Composition"
import { TextField, NumberField, SubmitButton } from '~our-app/ui-library'
// Мы также поддерживаем Valibot, ArkType и любые другие стандартные библиотеки схем
import { z } from 'zod'

const { fieldContext, formContext } = createFormHookContexts()

// Позволяет привязывать компоненты к форме для сохранения безопасности типов, но с уменьшением шаблонного кода в продакшене
// Определите это один раз, чтобы иметь генератор согласованных экземпляров форм во всем приложении
const { useAppForm } = createFormHook({
  fieldComponents: {
    TextField,
    NumberField,
  },
  formComponents: {
    SubmitButton,
  },
  fieldContext,
  formContext,
})

const PeoplePage = () => {
  const form = useAppForm({
    defaultValues: {
      username: '',
      age: 0,
    },
    validators: {
      // Передайте схему или функцию для валидации
      onChange: z.object({
        username: z.string(),
        age: z.number().min(13),
      }),
    },
    onSubmit: ({ value }) => {
      // Сделайте что-то с данными формы
      alert(JSON.stringify(value, null, 2))
    },
  })

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault()
        form.handleSubmit()
      }}
    >
      <h1>Personal Information</h1>
      {/* Компоненты привязаны к `form` и `field` для обеспечения максимальной безопасности типов */}
      {/* Используйте `form.AppField` для рендеринга компонента, привязанного к одному полю */}
      <form.AppField
        name="username"
        children={(field) => <field.TextField label="Full Name" />}
      />
      {/* Свойство "name" вызовет ошибку TypeScript, если будет опечатка */}
      <form.AppField
        name="age"
        children={(field) => <field.NumberField label="Age" />}
      />
      {/* Компоненты в `form.AppForm` имеют доступ к контексту формы */}
      <form.AppForm>
        <form.SubmitButton />
      </form.AppForm>
    </form>
  )
}

const rootElement = document.getElementById('root')!
ReactDOM.createRoot(rootElement).render(<PeoplePage />)
```

Хотя мы обычно рекомендуем использовать `createFormHook` для уменьшения шаблонного кода в долгосрочной перспективе, мы также поддерживаем одноразовые компоненты и другие поведения с помощью `useForm` и `form.Field`:

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { useForm } from '@tanstack/react-form'

const PeoplePage = () => {
  const form = useForm({
    defaultValues: {
      username: '',
      age: 0,
    },
    onSubmit: ({ value }) => {
      // Сделайте что-то с данными формы
      alert(JSON.stringify(value, null, 2))
    },
  })

  return (
    <form.Field
      name="age"
      validators={{
        // Мы можем выбирать между валидаторами для всей формы и для конкретного поля
        onChange: ({ value }) =>
          value > 13 ? undefined : 'Must be 13 or older',
      }}
      children={(field) => (
        <>
          <input
            name={field.name}
            value={field.state.value}
            onBlur={field.handleBlur}
            type="number"
            onChange={(e) => field.handleChange(e.target.valueAsNumber)}
          />
          {!field.state.meta.isValid && (
            <em>{field.state.meta.errors.join(',')}</em>
          )}
        </>
      )}
    />
  )
}

const rootElement = document.getElementById('root')!
ReactDOM.createRoot(rootElement).render(<PeoplePage />)
```

Все свойства из `useForm` можно использовать в `useAppForm`, и все свойства из `form.Field` можно использовать в `form.AppField`.
