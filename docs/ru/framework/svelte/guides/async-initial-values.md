---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T22:01:51.753Z'
id: async-initial-values
title: Асинхронные начальные значения
---

Допустим, вы хотите получить данные из API и использовать их как начальные значения формы.

Хотя эта задача кажется простой на первый взгляд, в ней скрыты сложности, о которых вы могли не задумываться до сих пор.

Например, вам может потребоваться показать индикатор загрузки во время получения данных или корректно обработать возможные ошибки.  
Аналогично, вы можете столкнуться с необходимостью кэширования данных, чтобы избежать их повторной загрузки при каждом рендеринге формы.

Хотя многие из этих функций можно реализовать с нуля, результат будет очень похож на другой наш проект: [TanStack Query](https://tanstack.com/query).

Поэтому в этом руководстве показано, как можно комбинировать TanStack Form и TanStack Query для достижения желаемого поведения.

## Базовое использование

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'
  import { createQuery } from '@tanstack/svelte-query'

    const { data, isLoading } = createQuery(() => ({
      queryKey: ['data'],
      queryFn: async () => {
        await new Promise((resolve) => setTimeout(resolve, 1000))
        return { firstName: 'FirstName', lastName: 'LastName' }
      },
    }))

    const form = createForm(() => ({
      defaultValues: {
        firstName: $data?.firstName ?? '',
        lastName: $data?.lastName ?? '',
      },
      onSubmit: async ({ value }) => {
        // Do something with form data
        console.log(value)
      },
    }))
</script>

{#if $isLoading}
  <p>Loading...</p>
{:else}
  <!-- form... -->
{/if}
```

Этот код отобразит индикатор загрузки до завершения получения данных, а затем выведет форму с загруженными данными в качестве начальных значений.
