---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:48:23.067Z'
id: async-initial-values
title: Valores iniciales asíncronos
---

# Valores Iniciales Asíncronos (Async Initial Values)

Digamos que desea obtener algunos datos de una API y utilizarlos como valor inicial de un formulario.

Aunque este problema parece simple en la superficie, existen complejidades ocultas en las que quizás no haya pensado hasta ahora.

Por ejemplo, es posible que desee mostrar un indicador de carga (spinner) mientras se obtienen los datos, o manejar los errores de manera elegante.

Asimismo, también podría encontrarse buscando una forma de almacenar en caché los datos para no tener que recuperarlos cada vez que se renderiza el formulario.

Si bien podríamos implementar muchas de estas características desde cero, terminaría pareciéndose mucho a otro proyecto que mantenemos: [TanStack Query](https://tanstack.com/query).

Por lo tanto, esta guía le muestra cómo puede combinar TanStack Form con TanStack Query para lograr el comportamiento deseado.

## Uso Básico

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
  <p v-if="isLoading">Cargando..</p>
  <form v-else @submit.prevent.stop="form.handleSubmit">
    <!-- ... -->
  </form>
</template>
```

Esto mostrará un texto de carga hasta que se obtengan los datos, y luego renderizará el formulario con los datos recuperados como valores iniciales.
