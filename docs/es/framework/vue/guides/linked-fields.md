---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:48:44.603Z'
id: linked-fields
title: Campos vinculados
---

## Vincular Dos Campos de Formulario Juntos

Puede encontrarse en la necesidad de vincular dos campos juntos; cuando uno se valida en función de que el valor de otro campo haya cambiado.  
Un caso de uso común es cuando tiene tanto un campo `password` como un campo `confirm_password`,  
donde desea que `confirm_password` muestre un error cuando su valor no coincida con el de `password`,  
sin importar qué campo haya desencadenado el cambio de valor.

Imagine el siguiente flujo de usuario:

- El usuario actualiza el campo de confirmación de contraseña.
- El usuario actualiza el campo de contraseña principal.

En este ejemplo, el formulario seguirá mostrando errores,  
ya que la validación del campo "confirmar contraseña" no se ha vuelto a ejecutar para marcarlo como aceptado.

Para solucionar esto, debemos asegurarnos de que la validación de "confirmar contraseña" se re-ejecute cuando se actualice el campo de contraseña.  
Para lograrlo, puede agregar la propiedad `onChangeListenTo` al campo `confirm_password`.

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    password: '',
    confirm_password: '',
  },
  // ...
})
</script>

<template>
  <div>
    <form @submit.prevent.stop="form.handleSubmit">
      <div>
        <form.Field name="password">
          <template v-slot="{ field }">
            <div>Contraseña:</div>
            <input
              :value="field.state.value"
              @input="
                (e) => field.handleChange((e.target as HTMLInputElement).value)
              "
            />
          </template>
        </form.Field>
        <form.Field
          name="confirm_password"
          :validators="{
            onChangeListenTo: ['password'],
            onChange: ({ value, fieldApi }) => {
              if (value !== fieldApi.form.getFieldValue('password')) {
                return 'Las contraseñas no coinciden'
              }
              return undefined
            },
          }"
        >
          <template v-slot="{ field }">
            <div>Confirmar Contraseña:</div>
            <input
              :value="field.state.value"
              @input="
                (e) => field.handleChange((e.target as HTMLInputElement).value)
              "
            />
            <div v-for="(err, index) in field.state.meta.errors" :key="index">
              {{ err }}
            </div>
          </template>
        </form.Field>
      </div>
      <button type="submit">Enviar</button>
    </form>
  </div>
</template>
```

Esto también funciona con la propiedad `onBlurListenTo`, que re-ejecutará la validación cuando el campo pierda el foco.
