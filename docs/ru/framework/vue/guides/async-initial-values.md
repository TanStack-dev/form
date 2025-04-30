---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:59:57.669Z'
id: async-initial-values
title: Асинхронные начальные значения
---

Предположим, вы хотите получить данные из API и использовать их как начальные значения формы.

Хотя эта задача кажется простой на первый взгляд, существуют скрытые сложности, о которых вы могли не задумываться.

Например, вы можете захотеть отображать индикатор загрузки во время получения данных или корректно обрабатывать возможные ошибки.

Аналогично, может возникнуть необходимость кэшировать данные, чтобы избежать повторных запросов при каждом рендеринге формы.

Хотя многие из этих функций можно реализовать с нуля, результат будет напоминать другой наш проект: [TanStack Query](https://tanstack.com/query).

Поэтому в этом руководстве мы покажем, как можно комбинировать TanStack Form с TanStack Query для достижения желаемого поведения.

## Базовое использование

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'
import { useQuery } from '@tanstack/vue-query'
import { watchEffect, reactive } from 'vue'

const { data, isLoading } = useQuery({
  queryKey: ['data'],
  queryFn: async () => {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return { firstName: 'FirstName', lastName: 'LastName' }
  },
})

const firstName = computed(() => data.value?.firstName || '')
const lastName = computed(() => data.value?.lastName || '')

const defaultValues = reactive({
  firstName,
  lastName,
})

const form = useForm({
  defaultValues,
  onSubmit: async ({ value }) => {
    // Do something with form data
    console.log(value)
  },
})
</script>

<template>
  <p v-if="isLoading">Loading..</p>
  <form v-else @submit.prevent.stop="form.handleSubmit">
    <!-- ... -->
  </form>
</template>
```

Этот код отображает текст "Loading.." до завершения загрузки данных, а затем рендерит форму с полученными данными в качестве начальных значений.
