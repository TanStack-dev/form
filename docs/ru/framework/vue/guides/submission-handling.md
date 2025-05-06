---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:26:15.936Z'
id: submission-handling
title: Submission handling
---

## Передача дополнительных данных в обработку отправки формы

У вас может быть несколько типов поведения при отправке формы, например, возврат на другую страницу или остаться на форме.  
Это можно реализовать, указав свойство `onSubmitMeta`. Эти метаданные будут переданы в функцию `onSubmit`.

> Примечание: если `form.handleSubmit()` вызывается без метаданных, будут использованы указанные значения по умолчанию.

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null
}

// Метаданные не обязательны для вызова form.handleSubmit().
// Укажите, какие значения использовать по умолчанию, если метаданные не переданы
const defaultMeta: FormMeta = {
  submitAction: null,
}

const form = useForm({
  defaultValues: {
    data: '',
  },
  // Определите, какие метаданные ожидать при отправке
  onSubmitMeta: defaultMeta,
  onSubmit: async ({ value, meta }) => {
    // Сделайте что-то с переданными значениями через handleSubmit
    console.log(`Выбранное действие - ${meta.submitAction}`, value)
  },
})
</script>

<template>
  <form @submit.prevent.stop="">
    <button
      type="submit"
      @click="
        () => {
          // Переопределяет значение по умолчанию, указанное в onSubmitMeta
          form.handleSubmit({ submitAction: 'continue' })
        }
      "
    >
      Отправить и продолжить
    </button>
    <button
      type="submit"
      @click="() => form.handleSubmit({ submitAction: 'backToMenu' })"
    >
      Отправить и вернуться в меню
    </button>
  </form>
</template>
```

## Преобразование данных с помощью стандартных схем (Standard Schemas)

Хотя Tanstack Form предоставляет [поддержку стандартных схем (Standard Schema)](./validation.md) для валидации, он не сохраняет выходные данные схемы.

Значение, передаваемое в функцию `onSubmit`, всегда будет входными данными. Чтобы получить выходные данные стандартной схемы, обработайте их в функции `onSubmit`:

```vue
<script setup lang="ts">
import { z } from 'zod'
import { useForm } from '@tanstack/vue-form'

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
    // Обработайте данные через схему, чтобы получить преобразованное значение
    const result = schema.parse(value)
    const outputAge: number = result.age
  },
})
</script>
<template>
  <!-- ... -->
</template>
```
