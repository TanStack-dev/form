---
source-updated-at: '2025-04-27T15:54:55.000Z'
translation-updated-at: '2025-04-30T21:48:02.638Z'
id: listeners
title: Escuchadores
---

Para situaciones donde desee "afectar" o "reaccionar" a activadores, existe la API de escucha (listener API). Por ejemplo, si usted, como desarrollador, desea restablecer un campo del formulario como resultado del cambio de otro campo, utilizaría la API de escucha.

Imagine el siguiente flujo de usuario:

- El usuario selecciona un país de un menú desplegable.
- Luego, el usuario selecciona una provincia de otro menú desplegable.
- El usuario cambia el país seleccionado por uno diferente.

En este ejemplo, cuando el usuario cambia el país, la provincia seleccionada debe restablecerse ya que ya no es válida. Con la API de escucha, podemos suscribirnos al evento `onChange` y despachar un restablecimiento al campo "province" cuando se active el listener.

Los eventos que pueden "escucharse" son:

- `onChange`
- `onBlur`
- `onMount`
- `onSubmit`

```tsx
function App() {
  const form = useForm({
    defaultValues: {
      country: '',
      province: '',
    },
    // ...
  })

  return (
    <div>
      <form.Field
        name="country"
        listeners={{
          onChange: ({ value }) => {
            console.log(`Country changed to: ${value}, resetting province`)
            form.setFieldValue('province', '')
          },
        }}
      >
        {(field) => (
          <label>
            <div>Country</div>
            <input
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
          </label>
        )}
      </form.Field>

      <form.Field name="province">
        {(field) => (
          <label>
            <div>Province</div>
            <input
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
          </label>
        )}
      </form.Field>
    </div>
  )
}
```

### Debounce incorporado

Si está realizando una solicitud API dentro de un listener, puede que desee aplicar debounce a las llamadas, ya que puede causar problemas de rendimiento.  
Habilitamos un método sencillo para aplicar debounce a sus listeners agregando `onChangeDebounceMs` u `onBlurDebounceMs`.

```tsx
<form.Field
  name="country"
  listeners={{
    onChangeDebounceMs: 500, // debounce de 500ms
    onChange: ({ value }) => {
      console.log(`Country changed to: ${value} without a change within 500ms, resetting province`)
      form.setFieldValue('province', '')
    },
  }}
>
  {(field) => (
    /* ... */
  )}
</form.Field>
```

### Listeners del formulario

A un nivel superior, los listeners también están disponibles a nivel del formulario, lo que le permite acceder a los eventos `onMount` y `onSubmit`, y propagar `onChange` y `onBlur` a todos los campos hijos del formulario. Los listeners a nivel del formulario también pueden tener debounce aplicado de la misma manera que se mencionó anteriormente.

Los listeners `onMount` y `onSubmit` tienen las siguientes propiedades:

- `formApi`

Los listeners `onChange` y `onBlur` tienen acceso a:

- `fieldApi`
- `formApi`

```tsx
const form = useForm({
  listeners: {
    onMount: ({ formApi }) => {
      // servicio de registro personalizado
      loggingService('mount', formApi.state.values)
    },

    onChange: ({ formApi, fieldApi }) => {
      // lógica de guardado automático
      if (formApi.state.isValid) {
        formApi.handleSubmit()
      }

      // fieldApi representa el campo que activó el evento.
      console.log(fieldApi.name, fieldApi.state.value)
    },
    onChangeDebounceMs: 500,
  },
})
```
