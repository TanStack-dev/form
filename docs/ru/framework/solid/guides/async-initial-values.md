---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T22:01:21.066Z'
id: async-initial-values
title: Асинхронные начальные значения
---

Допустим, вы хотите получить данные из API и использовать их в качестве начальных значений формы.

Хотя эта задача кажется простой на первый взгляд, она скрывает сложности, о которых вы могли не задумываться ранее.

Например, вы можете захотеть показать индикатор загрузки во время получения данных или корректно обработать возможные ошибки.  
Аналогично, может возникнуть необходимость кэшировать данные, чтобы избежать повторных запросов при каждом рендеринге формы.

Хотя многие из этих функций можно реализовать с нуля, результат будет напоминать другой наш проект: [TanStack Query](https://tanstack.com/query).

Поэтому в этом руководстве мы покажем, как можно комбинировать TanStack Form и TanStack Query для достижения нужного поведения.

## Базовое использование

```tsx
import { createForm } from '@tanstack/solid-form'
import { createQuery } from '@tanstack/solid-query'

export default function App() {
  const { data, isLoading } = createQuery(() => ({
    queryKey: ['data'],
    queryFn: async () => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return { firstName: 'FirstName', lastName: 'LastName' }
    },
  }))

  const form = createForm(() => ({
    defaultValues: {
      firstName: data?.firstName ?? '',
      lastName: data?.lastName ?? '',
    },
    onSubmit: async ({ value }) => {
      // Do something with form data
      console.log(value)
    },
  }))

  if (isLoading) return <p>Loading..</p>

  return null
}
```

Этот код покажет индикатор загрузки до завершения запроса, а затем отрендерит форму с полученными данными в качестве начальных значений.
