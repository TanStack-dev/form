---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:59:02.025Z'
id: async-initial-values
title: Асинхронные начальные значения
---

Допустим, вы хотите получить данные из API и использовать их в качестве начальных значений формы.

Хотя эта задача кажется простой на первый взгляд, она скрывает сложности, о которых вы могли не задумываться ранее.

Например, вы можете захотеть отображать индикатор загрузки во время получения данных или корректно обрабатывать ошибки.  
Аналогично, может возникнуть необходимость кэшировать данные, чтобы избежать их повторной загрузки при каждом рендеринге формы.

Хотя многие из этих функций можно реализовать с нуля, в итоге это будет похоже на другой наш проект: [TanStack Query](https://tanstack.com/query).

Поэтому в этом руководстве мы покажем, как можно комбинировать TanStack Form с TanStack Query для достижения желаемого поведения.

## Базовое использование

```tsx
import { useForm } from '@tanstack/react-form'
import { useQuery } from '@tanstack/react-query'

export default function App() {
  const {data, isLoading} = useQuery({
    queryKey: ['data'],
    queryFn: async () => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return {firstName: 'FirstName', lastName: "LastName"}
    }
  })

  const form = useForm({
    defaultValues: {
      firstName: data?.firstName ?? '',
      lastName: data?.lastName ?? '',
    },
    onSubmit: async ({ value }) => {
      // Do something with form data
      console.log(value)
    },
  })

  if (isLoading) return <p>Loading..</p>

  return (
    // ...
  )
}
```

Этот код отобразит индикатор загрузки до получения данных, а затем выведет форму с полученными данными в качестве начальных значений.
