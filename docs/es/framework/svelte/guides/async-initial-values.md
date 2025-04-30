---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:50:17.278Z'
id: async-initial-values
title: Valores iniciales asíncronos
---

## Valores Iniciales Asíncronos

Digamos que desea obtener algunos datos de una API y utilizarlos como valor inicial de un formulario.

Aunque este problema parece simple en la superficie, existen complejidades ocultas en las que quizás no haya pensado hasta ahora.

Por ejemplo, es posible que desee mostrar un indicador de carga mientras se obtienen los datos, o manejar los errores de manera elegante.  
Asimismo, también podría encontrarse buscando una forma de almacenar en caché los datos para no tener que recuperarlos cada vez que se renderice el formulario.

Si bien podríamos implementar muchas de estas características desde cero, el resultado terminaría pareciéndose mucho a otro proyecto que mantenemos: [TanStack Query](https://tanstack.com/query).

Por lo tanto, esta guía le muestra cómo puede combinar TanStack Form con TanStack Query para lograr el comportamiento deseado.

## Uso Básico

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

Esto mostrará un indicador de carga hasta que se obtengan los datos, y luego renderizará el formulario con los datos recuperados como valores iniciales.
