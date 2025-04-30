---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:49:53.858Z'
id: linked-fields
title: Campos vinculados
---

## Vincular Dos Campos de Formulario

Puede que necesite vincular dos campos juntos; cuando uno se valida porque el valor de otro campo ha cambiado.  
Un caso de uso común es cuando tiene un campo `password` y otro `confirm_password`,  
donde desea que `confirm_password` muestre un error cuando su valor no coincida con `password`,  
sin importar qué campo haya desencadenado el cambio de valor.

Imagine el siguiente flujo de usuario:

- El usuario actualiza el campo de confirmación de contraseña.
- El usuario actualiza el campo de contraseña principal.

En este ejemplo, el formulario seguirá mostrando errores,  
ya que la validación del campo "confirmar contraseña" no se ha vuelto a ejecutar para marcarlo como aceptado.

Para solucionar esto, debemos asegurarnos de que la validación de "confirmar contraseña" se vuelva a ejecutar cuando se actualice el campo de contraseña.  
Para hacerlo, puede agregar la propiedad `onChangeListenTo` al campo `confirm_password`.

```tsx
export default function App() {
  const form = createForm(() => ({
    defaultValues: {
      password: '',
      confirm_password: '',
    },
    // ...
  }))

  return (
    <div>
      <form.Field name="password">
        {(field) => (
          <label>
            <div>Password</div>
            <input
              value={field().state.value}
              onChange={(e) => field().handleChange(e.target.value)}
            />
          </label>
        )}
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
        {(field) => (
          <div>
            <label>
              <div>Confirm Password</div>
              <input
                value={field().state.value}
                onChange={(e) => field().handleChange(e.target.value)}
              />
            </label>
            <Index each={field().state.meta.errors}>
              {(err) => <div>{err()}</div>}
            </Index>
          </div>
        )}
      </form.Field>
    </div>
  )
}
```

Esto también funciona con la propiedad `onBlurListenTo`, que volverá a ejecutar la validación cuando el campo pierda el foco.
