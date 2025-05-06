---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-05-06T20:19:49.980Z'
id: listeners
title: Side effects for event triggers
---

Para situaciones donde desee "afectar" o "reaccionar" a activadores, existe la API de escucha (listener API). Por ejemplo, si usted, como desarrollador, desea restablecer un campo del formulario como resultado de un cambio en otro campo, utilizaría la API de escucha.

Imagine el siguiente flujo de usuario:

- El usuario selecciona un país de un menú desplegable.
- Luego, el usuario selecciona una provincia de otro menú desplegable.
- El usuario cambia el país seleccionado por otro diferente.

En este ejemplo, cuando el usuario cambia el país, la provincia seleccionada debe restablecerse ya que ya no es válida. Con la API de escucha, podemos suscribirnos al evento `onChange` y enviar un restablecimiento al campo "province" cuando se active el escucha.

Los eventos que se pueden "escuchar" son:

- `onChange`
- `onBlur`
- `onMount`
- `onSubmit`

```vue
<script setup>
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    country: '',
    province: '',
  },
  // ...
})
</script>

<template>
  <div>
    <form.Field
      name="country"
      :listeners="{
        onChange: ({ value }) => {
          console.log(`Country changed to: ${value}, resetting province`)
          form.setFieldValue('province', '')
        },
      }"
    >
      <template v-slot="{ field }">
        <input
          :value="field.state.value"
          @input="(e) => field.handleChange(e.target.value)"
        />
      </template>
    </form.Field>

    <form.Field name="province">
      <template v-slot="{ field }">
        <input
          :value="field.state.value"
          @input="(e) => field.handleChange(e.target.value)"
        />
      </template>
    </form.Field>
  </div>
</template>
```
