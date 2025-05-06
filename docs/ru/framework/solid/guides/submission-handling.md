---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:27:15.819Z'
id: submission-handling
title: Submission handling
---

## Передача дополнительных данных при обработке отправки формы

У вас могут быть различные типы поведения при отправке формы, например, возврат на другую страницу или оставаться на форме.  
Этого можно добиться, указав свойство `onSubmitMeta`. Эти метаданные будут переданы в функцию `onSubmit`.

> Примечание: если `form.handleSubmit()` вызывается без метаданных, будут использованы значения по умолчанию.

```tsx
import { createForm } from '@tanstack/solid-form'

type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null
}

// Метаданные не обязательны для вызова form.handleSubmit().
// Укажите значения по умолчанию, если метаданные не переданы
const defaultMeta: FormMeta = {
  submitAction: null,
}

export default function App() {
  const form = createForm(() => ({
    defaultValues: {
      data: '',
    },
    // Определите, какие метаданные ожидаются при отправке
    onSubmitMeta: defaultMeta,
    onSubmit: async ({ value, meta }) => {
      // Сделайте что-то с переданными значениями через handleSubmit
      console.log(`Выбранное действие - ${meta.submitAction}`, value)
    },
  }))

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault()
        e.stopPropagation()
      }}
    >
      {/* ... */}
      <button
        type="submit"
        // Переопределяет значение по умолчанию из onSubmitMeta
        onClick={() => form.handleSubmit({ submitAction: 'continue' })}
      >
        Отправить и продолжить
      </button>
      <button
        type="submit"
        onClick={() => form.handleSubmit({ submitAction: 'backToMenu' })}
      >
        Отправить и вернуться в меню
      </button>
    </form>
  )
}
```

## Преобразование данных с помощью стандартных схем (Standard Schemas)

Хотя Tanstack Form поддерживает [стандартные схемы (Standard Schemas)](./validation.md) для валидации, он не сохраняет выходные данные схемы.

Значение, передаваемое в функцию `onSubmit`, всегда будет входными данными. Чтобы получить выходные данные стандартной схемы, обработайте их в функции `onSubmit`:

```tsx
import { createForm } from '@tanstack/solid-form'
import { z } from 'zod'

// ...

const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form использует входной тип стандартных схем
const defaultValues: z.input<typeof schema> = {
  age: '13',
}

const form = createForm(() => ({
  defaultValues,
  validators: {
    onChange: schema,
  },
  onSubmit: ({ value }) => {
    const inputAge: string = value.age
    // Обработайте данные через схему, чтобы получить преобразованное значение
    const result = schema.parse(value)
    const outputAge: number = result.age
  },
}))
```
