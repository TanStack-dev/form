---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T21:59:12.695Z'
id: overview
title: Обзор
---

# Обзор

TanStack Form — это лучшее решение для работы с формами в веб-приложениях, предлагающее мощный и гибкий подход к управлению формами. Благодаря первоклассной поддержке TypeScript, headless-компонентам (headless UI) и независимому от фреймворков дизайну, он упрощает обработку форм и обеспечивает бесшовный опыт работы с различными фронтенд-фреймворками.

## Мотивация

Большинство веб-фреймворков не предоставляют комплексного решения для работы с формами, вынуждая разработчиков создавать собственные реализации или полагаться на менее функциональные библиотеки. Это часто приводит к отсутствию согласованности, низкой производительности и увеличению времени разработки. TanStack Form призван решить эти проблемы, предлагая универсальное решение для управления формами, которое одновременно мощное и простое в использовании.

С TanStack Form разработчики могут решать распространённые задачи, связанные с формами, такие как:

- Реактивное связывание данных и управление состоянием (state management)
- Сложная валидация и обработка ошибок
- Доступность (accessibility) и адаптивный дизайн (responsive design)
- Интернационализация (internationalization) и локализация (localization)
- Кросс-платформенная совместимость и кастомизация стилей

Предоставляя полное решение для этих задач, TanStack Form позволяет разработчикам легко создавать надёжные и удобные формы.

## Хватит слов, покажите код!

В примере ниже показана работа TanStack Form с адаптером для фреймворка React:

[Открыть в CodeSandbox](https://codesandbox.io/s/github/tanstack/form/tree/main/examples/react/simple)

```tsx
import * as React from 'react'
import { createRoot } from 'react-dom/client'
import { useForm } from '@tanstack/react-form'
import type { AnyFieldApi } from '@tanstack/react-form'

function FieldInfo({ field }: { field: AnyFieldApi }) {
  return (
    <>
      {field.state.meta.isTouched && !field.state.meta.isValid ? (
        <em>{field.state.meta.errors.join(', ')}</em>
      ) : null}
      {field.state.meta.isValidating ? 'Validating...' : null}
    </>
  )
}

export default function App() {
  const form = useForm({
    defaultValues: {
      firstName: '',
      lastName: '',
    },
    onSubmit: async ({ value }) => {
      // Do something with form data
      console.log(value)
    },
  })

  return (
    <div>
      <h1>Simple Form Example</h1>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          form.handleSubmit()
        }}
      >
        <div>
          {/* A type-safe field component*/}
          <form.Field
            name="firstName"
            validators={{
              onChange: ({ value }) =>
                !value
                  ? 'A first name is required'
                  : value.length < 3
                    ? 'First name must be at least 3 characters'
                    : undefined,
              onChangeAsyncDebounceMs: 500,
              onChangeAsync: async ({ value }) => {
                await new Promise((resolve) => setTimeout(resolve, 1000))
                return (
                  value.includes('error') && 'No "error" allowed in first name'
                )
              },
            }}
            children={(field) => {
              // Avoid hasty abstractions. Render props are great!
              return (
                <>
                  <label htmlFor={field.name}>First Name:</label>
                  <input
                    id={field.name}
                    name={field.name}
                    value={field.state.value}
                    onBlur={field.handleBlur}
                    onChange={(e) => field.handleChange(e.target.value)}
                  />
                  <FieldInfo field={field} />
                </>
              )
            }}
          />
        </div>
        <div>
          <form.Field
            name="lastName"
            children={(field) => (
              <>
                <label htmlFor={field.name}>Last Name:</label>
                <input
                  id={field.name}
                  name={field.name}
                  value={field.state.value}
                  onBlur={field.handleBlur}
                  onChange={(e) => field.handleChange(e.target.value)}
                />
                <FieldInfo field={field} />
              </>
            )}
          />
        </div>
        <form.Subscribe
          selector={(state) => [state.canSubmit, state.isSubmitting]}
          children={([canSubmit, isSubmitting]) => (
            <button type="submit" disabled={!canSubmit}>
              {isSubmitting ? '...' : 'Submit'}
            </button>
          )}
        />
      </form>
    </div>
  )
}

const rootElement = document.getElementById('root')!

createRoot(rootElement).render(<App />)
```

## Вы меня убедили, что дальше?

- Изучите TanStack Form в удобном темпе с нашим подробным [Руководством](../installation) и [Справочником API](../reference/classes/formapi)
