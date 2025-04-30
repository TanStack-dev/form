---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-04-30T21:59:38.024Z'
id: submission-handling
title: Обработка отправки
---

## Передача дополнительных данных в обработку отправки формы

У вас может быть несколько типов поведения при отправке формы, например, возврат на другую страницу или остаться на форме.  
Этого можно достичь, указав свойство `onSubmitMeta`. Эти метаданные будут переданы в функцию `onSubmit`.

> Примечание: если `form.handleSubmit()` вызывается без метаданных, будут использованы указанные значения по умолчанию.

```tsx
import { useForm } from '@tanstack/react-form'

type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null
}

// Метаданные не обязательны для вызова form.handleSubmit().
// Укажите значения по умолчанию, если метаданные не переданы
const defaultMeta: FormMeta = {
  submitAction: null,
}

function App() {
  const form = useForm({
    defaultValues: {
      data: '',
    },
    // Определите, какие метаданные ожидать при отправке
    onSubmitMeta: defaultMeta,
    onSubmit: async ({ value, meta }) => {
      // Сделайте что-то с переданными значениями через handleSubmit
      console.log(`Selected action - ${meta.submitAction}`, value)
    },
  })

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

Хотя Tanstack Form предоставляет [поддержку стандартных схем (Standard Schema)](./validation.md) для валидации, он не сохраняет выходные данные схемы.

Значение, передаваемое в функцию `onSubmit`, всегда будет входными данными. Чтобы получить выходные данные стандартной схемы, обработайте их в функции `onSubmit`:

```tsx
const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form использует входной тип стандартных схем
const defaultValues: z.input<typeof schema> = {
  age: '13',
}

const form = useForm({
  defaultValues,
  validators: {
    onChange: schema,
  },
  onSubmit: ({ value }) => {
    const inputAge: string = value.age
    // Пропустите данные через схему, чтобы получить преобразованное значение
    const result = schema.parse(value)
    const outputAge: number = result.age
  },
})
```
