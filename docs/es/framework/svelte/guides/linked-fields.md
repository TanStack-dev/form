---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:50:30.314Z'
id: linked-fields
title: Campos vinculados
---

Puede que necesite vincular dos campos de formulario, donde uno se valide cuando el valor del otro campo haya cambiado.  
Un caso de uso común es cuando tiene un campo `password` y otro `confirm_password`,  
donde desea que `confirm_password` muestre un error cuando su valor no coincida con `password`,  
sin importar qué campo haya desencadenado el cambio de valor.

Imagine el siguiente flujo de usuario:

- El usuario actualiza el campo de confirmación de contraseña.
- El usuario actualiza el campo de contraseña principal.

En este ejemplo, el formulario seguirá mostrando errores,  
ya que la validación del campo "confirmar contraseña" no se ha vuelto a ejecutar para marcarlo como válido.

Para solucionarlo, debemos asegurarnos de que la validación de "confirmar contraseña" se vuelva a ejecutar cuando se actualice el campo de contraseña principal.  
Para ello, puede agregar la propiedad `onChangeListenTo` al campo `confirm_password`.

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      password: '',
      confirm_password: '',
    },
    // ...
  }))
</script>

<div>
  <form.Field name="password">
    {#snippet children(field)}
      <label>
        <div>Password</div>
        <input
          value={field.state.value}
          onChange={(e) => field.handleChange(e.target.value)}
        />
      </label>
    {/snippet}
  </form.Field>
  <form.Field
    name="confirm_password"
    validators={{
      onChangeListenTo: ['password'],
      onChange: ({ value, fieldApi }) => {
        if (value !== fieldApi.form.getFieldValue('password')) {
          return 'Passwords do not match'
        }
        return undefined
      },
    }}
  >
    {#snippet children(field)}
      <div>
        <label>
          <div>Confirm Password</div>
          <input
            value={field.state.value}
            onchange={(e) => field.handleChange(e.target.value)}
          />
        </label>
        {#each field.state.meta.errors as err}
          <div>{err}</div>
        {/each}
      </div>
    {/snippet}
  </form.Field>
</div>
```

Esto también funciona con la propiedad `onBlurListenTo`, que volverá a ejecutar la validación cuando el campo pierda el foco.
