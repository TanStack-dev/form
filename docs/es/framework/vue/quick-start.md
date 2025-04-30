---
source-updated-at: '2025-03-31T17:05:18.000Z'
translation-updated-at: '2025-04-30T21:47:10.809Z'
id: quick-start
title: Inicio rápido
---

## Motivación

La mayoría de los frameworks web no ofrecen una solución integral para el manejo de formularios, lo que obliga a los desarrolladores a crear implementaciones personalizadas o depender de bibliotecas menos capaces. Esto a menudo resulta en falta de consistencia, bajo rendimiento y mayor tiempo de desarrollo. TanStack Form busca abordar estos desafíos proporcionando una solución todo en uno para gestionar formularios que es tanto potente como fácil de usar.

Con TanStack Form, los desarrolladores pueden abordar desafíos comunes relacionados con formularios, tales como:

- Enlace de datos reactivos y gestión de estado
- Validación compleja y manejo de errores
- Accesibilidad y diseño responsivo
- Internacionalización y localización
- Compatibilidad multiplataforma y estilos personalizados

Al proporcionar una solución completa para estos desafíos, TanStack Form permite a los desarrolladores construir formularios robustos y fáciles de usar con facilidad.

Lo mínimo indispensable para comenzar con TanStack Form es crear un formulario y agregar un campo. Tenga en cuenta que este ejemplo no incluye validación ni manejo de errores... por ahora.

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

A partir de aquí, estará listo para explorar todas las demás características de TanStack Form.
