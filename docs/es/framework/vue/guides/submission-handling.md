---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:19:59.902Z'
id: submission-handling
title: Submission handling
---

## Pasar datos adicionales al manejo de envíos

Puede que tengas múltiples tipos de comportamiento al enviar un formulario, por ejemplo, volver a otra página o permanecer en el formulario.  
Puedes lograr esto especificando la propiedad `onSubmitMeta`. Estos metadatos se pasarán a la función `onSubmit`.

> Nota: si `form.handleSubmit()` se llama sin metadatos, se utilizarán los valores predeterminados proporcionados.

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null
}

// No se requieren metadatos para llamar a form.handleSubmit().
// Especifica qué valores usar como predeterminados si no se pasan metadatos
const defaultMeta: FormMeta = {
  submitAction: null,
}

const form = useForm({
  defaultValues: {
    data: '',
  },
  // Define qué valores de metadatos esperar al enviar
  onSubmitMeta: defaultMeta,
  onSubmit: async ({ value, meta }) => {
    // Haz algo con los valores pasados a través de handleSubmit
    console.log(`Selected action - ${meta.submitAction}`, value)
  },
})
</script>

<template>
  <form @submit.prevent.stop="">
    <button
      type="submit"
      @click="
        () => {
          // Sobrescribe el valor predeterminado especificado en onSubmitMeta
          form.handleSubmit({ submitAction: 'continue' })
        }
      "
    >
      Enviar y continuar
    </button>
    <button
      type="submit"
      @click="() => form.handleSubmit({ submitAction: 'backToMenu' })"
    >
      Enviar y volver al menú
    </button>
  </form>
</template>
```

## Transformar datos con Esquemas Estándar

Si bien Tanstack Form ofrece [soporte para Esquemas Estándar](./validation.md) para validación, no conserva los datos de salida del Esquema.

El valor pasado a la función `onSubmit` siempre serán los datos de entrada. Para recibir los datos de salida de un Esquema Estándar, analízalos en la función `onSubmit`:

```vue
<script setup lang="ts">
import { z } from 'zod'
import { useForm } from '@tanstack/vue-form'

const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form usa el tipo de entrada de los Esquemas Estándar
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
    // Pásalo por el esquema para obtener el valor transformado
    const result = schema.parse(value)
    const outputAge: number = result.age
  },
})
</script>
<template>
  <!-- ... -->
</template>
```
