---
source-updated-at: '2025-03-31T17:05:18.000Z'
translation-updated-at: '2025-04-30T21:58:39.065Z'
id: quick-start
title: Быстрый старт
---

Минимальный набор для начала работы с TanStack Form — это создание формы и добавление поля. Обратите внимание, что в этом примере пока нет валидации или обработки ошибок... пока что.

```vue
<!-- App.vue -->
<script setup>
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    fullName: '',
  },
  onSubmit: async ({ value }) => {
    // Do something with form data
    console.log(value)
  },
})
</script>

<template>
  <div>
    <form @submit.prevent.stop="form.handleSubmit">
      <div>
        <form.Field name="fullName">
          <template v-slot="{ field }">
            <input
              :name="field.name"
              :value="field.state.value"
              @blur="field.handleBlur"
              @input="(e) => field.handleChange((e.target as HTMLInputElement).value)"
            />
          </template>
        </form.Field>
      </div>
      <button type="submit">Submit</button>
    </form>
  </div>
</template>
```

Отсюда вы сможете перейти к изучению всех остальных возможностей TanStack Form!
