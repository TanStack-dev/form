---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:47:30.040Z'
id: async-initial-values
title: Valores iniciales asíncronos
---

## Valores Iniciales Asíncronos

Digamos que desea obtener algunos datos de una API y utilizarlos como valor inicial de un formulario.

Aunque este problema parece simple en la superficie, existen complejidades ocultas que quizás no haya considerado hasta ahora.

Por ejemplo, es posible que desee mostrar un indicador de carga mientras se obtienen los datos o manejar los errores de manera elegante.  
Asimismo, también podría encontrarse buscando una forma de almacenar en caché los datos para no tener que recuperarlos cada vez que se renderice el formulario.

Si bien podríamos implementar muchas de estas características desde cero, terminaría pareciéndose mucho a otro proyecto que mantenemos: [TanStack Query](https://tanstack.com/query).

Por lo tanto, esta guía le muestra cómo puede combinar TanStack Form con TanStack Query para lograr el comportamiento deseado.

## Uso Básico

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

Esto mostrará un indicador de carga hasta que se obtengan los datos y luego renderizará el formulario con los datos recuperados como valores iniciales.
