---
source-updated-at: '2025-04-16T08:45:06.000Z'
translation-updated-at: '2025-04-30T21:48:43.089Z'
id: basic-concepts
title: Conceptos básicos
---

# Conceptos Básicos y Terminología

Esta página introduce los conceptos básicos y la terminología utilizada en la biblioteca `@tanstack/react-form`. Familiarizarse con estos conceptos le ayudará a comprender y trabajar mejor con la biblioteca.

## Opciones del Formulario

Puede crear opciones para su formulario para que puedan compartirse entre múltiples formularios utilizando la función `formOptions`.

Ejemplo:

```tsx
interface User {
  firstName: string
  lastName: string
  hobbies: Array<string>
}
const defaultUser: User = { firstName: '', lastName: '', hobbies: [] }

const formOpts = formOptions({
  defaultValues: defaultUser,
})
```

## Instancia del Formulario

Una Instancia de Formulario es un objeto que representa un formulario individual y proporciona métodos y propiedades para trabajar con el formulario. Se crea una instancia de formulario utilizando el hook `useForm` proporcionado por las opciones del formulario. El hook acepta un objeto con una función `onSubmit`, que se llama cuando se envía el formulario.

```tsx
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // Hacer algo con los datos del formulario
    console.log(value)
  },
})
```

También puede crear una instancia de formulario sin usar `formOptions` utilizando la API independiente `useForm`:

```tsx
interface User {
  firstName: string
  lastName: string
  hobbies: Array<string>
}
const defaultUser: User = { firstName: '', lastName: '', hobbies: [] }

const form = useForm({
  defaultValues: defaultUser,
  onSubmit: async ({ value }) => {
    // Hacer algo con los datos del formulario
    console.log(value)
  },
})
```

## Campo

Un Campo representa un único elemento de entrada de formulario, como un campo de texto o una casilla de verificación. Los campos se crean utilizando el componente `form.Field` proporcionado por la instancia del formulario. El componente acepta una propiedad `name`, que debe coincidir con una clave en los valores predeterminados del formulario. También acepta una propiedad `children`, que es una función de renderizado que recibe un objeto de campo como argumento.

Ejemplo:

```tsx
<form.Field
  name="firstName"
  children={(field) => (
    <>
      <input
        value={field.state.value}
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.value)}
      />
      <FieldInfo field={field} />
    </>
  )}
/>
```

## Estado del Campo

Cada campo tiene su propio estado, que incluye su valor actual, estado de validación, mensajes de error y otros metadatos. Puede acceder al estado de un campo utilizando la propiedad `field.state`.

Ejemplo:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Hay tres estados de campo que pueden ser útiles para ver cómo interactúa el usuario con un campo: un campo está _"touched"_ (tocado) cuando el usuario hace clic o tabula en él, _"pristine"_ (prístino) hasta que el usuario cambia su valor, y _"dirty"_ (modificado) después de que el valor ha sido cambiado. Puede verificar estos estados a través de las banderas `isTouched`, `isPristine` e `isDirty`, como se muestra a continuación.

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Estados del campo](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

> **Nota importante para usuarios que vienen de `React Hook Form`**: la bandera `isDirty` en `TanStack/form` es diferente de la bandera con el mismo nombre en RHF.
> En RHF, `isDirty = true`, cuando los valores del formulario son diferentes de los valores originales. Si el usuario cambia los valores en un formulario y luego los cambia nuevamente para terminar con valores que coinciden con los valores predeterminados del formulario, `isDirty` será `false` en RHF, pero `true` en `TanStack/form`.
> Los valores predeterminados se exponen tanto a nivel del formulario como del campo en `TanStack/form` (`form.options.defaultValues`, `field.options.defaultValue`), por lo que puede escribir su propia función auxiliar `isDefaultValue()` si necesita emular el comportamiento de RHF.

## API del Campo

La API del Campo es un objeto pasado a la función de renderizado al crear un campo. Proporciona métodos para trabajar con el estado del campo.

Ejemplo:

```tsx
<input
  value={field.state.value}
  onBlur={field.handleBlur}
  onChange={(e) => field.handleChange(e.target.value)}
/>
```

## Validación

`@tanstack/react-form` proporciona validación tanto síncrona como asíncrona de forma predeterminada. Las funciones de validación se pueden pasar al componente `form.Field` utilizando la propiedad `validators`.

Ejemplo:

```tsx
<form.Field
  name="firstName"
  validators={{
    onChange: ({ value }) =>
      !value
        ? 'Se requiere un nombre'
        : value.length < 3
          ? 'El nombre debe tener al menos 3 caracteres'
          : undefined,
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && 'No se permite "error" en el nombre'
    },
  }}
  children={(field) => (
    <>
      <input
        value={field.state.value}
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.value)}
      />
      <FieldInfo field={field} />
    </>
  )}
/>
```

## Validación con Bibliotecas de Esquema Estándar

Además de las opciones de validación personalizadas, también admitimos la especificación [Standard Schema](https://github.com/standard-schema/standard-schema).

Puede definir un esquema utilizando cualquiera de las bibliotecas que implementan la especificación y pasarlo a un validador de formulario o campo.

Las bibliotecas admitidas incluyen:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```tsx
import { z } from 'zod'

const userSchema = z.object({
  age: z.number().gte(13, 'Debes tener 13 años para crear una cuenta'),
})

function App() {
  const form = useForm({
    defaultValues: {
      age: 0,
    },
    validators: {
      onChange: userSchema,
    },
  })
  return (
    <div>
      <form.Field
        name="age"
        children={(field) => {
          return <>{/* ... */}</>
        }}
      />
    </div>
  )
}
```

## Reactividad

`@tanstack/react-form` ofrece varias formas de suscribirse a cambios de estado del formulario y de los campos, más notablemente el hook `useStore(form.store)` y el componente `form.Subscribe`. Estos métodos le permiten optimizar el rendimiento de renderizado de su formulario actualizando solo los componentes cuando sea necesario.

Ejemplo:

```tsx
const firstName = useStore(form.store, (state) => state.values.firstName)
//...
<form.Subscribe
  selector={(state) => [state.canSubmit, state.isSubmitting]}
  children={([canSubmit, isSubmitting]) => (
    <button type="submit" disabled={!canSubmit}>
      {isSubmitting ? '...' : 'Enviar'}
    </button>
  )}
/>
```

Es importante recordar que, aunque la propiedad `selector` del hook `useStore` es opcional, se recomienda encarecidamente proporcionar una, ya que omitirla resultará en re-renderizados innecesarios.

```tsx
// Uso correcto
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
// Uso incorrecto
const store = useStore(form.store)
```

Nota: Se desaconseja el uso del hook `useField` para lograr reactividad, ya que está diseñado para usarse cuidadosamente dentro del componente `form.Field`. Es posible que desee usar `useStore(form.store)` en su lugar.

## Oyentes (Listeners)

`@tanstack/react-form` le permite reaccionar a desencadenantes específicos y "escucharlos" para despachar efectos secundarios.

Ejemplo:

```tsx
<form.Field
  name="country"
  listeners={{
    onChange: ({ value }) => {
      console.log(`País cambiado a: ${value}, reiniciando provincia`)
      form.setFieldValue('province', '')
    },
  }}
/>
```

Más información se puede encontrar en [Oyentes](./listeners.md)

## Campos de Arreglo

Los campos de arreglo le permiten administrar una lista de valores dentro de un formulario, como una lista de pasatiempos. Puede crear un campo de arreglo utilizando el componente `form.Field` con la propiedad `mode="array"`.

Cuando trabaje con campos de arreglo, puede usar los métodos `pushValue`, `removeValue`, `swapValues` y `moveValue` de los campos para agregar, eliminar e intercambiar valores en el arreglo.

Ejemplo:

```tsx
<form.Field
  name="hobbies"
  mode="array"
  children={(hobbiesField) => (
    <div>
      Pasatiempos
      <div>
        {!hobbiesField.state.value.length
          ? 'No se encontraron pasatiempos.'
          : hobbiesField.state.value.map((_, i) => (
              <div key={i}>
                <form.Field
                  name={`hobbies[${i}].name`}
                  children={(field) => {
                    return (
                      <div>
                        <label htmlFor={field.name}>Nombre:</label>
                        <input
                          id={field.name}
                          name={field.name}
                          value={field.state.value}
                          onBlur={field.handleBlur}
                          onChange={(e) => field.handleChange(e.target.value)}
                        />
                        <button
                          type="button"
                          onClick={() => hobbiesField.removeValue(i)}
                        >
                          X
                        </button>
                        <FieldInfo field={field} />
                      </div>
                    )
                  }}
                />
                <form.Field
                  name={`hobbies[${i}].description`}
                  children={(field) => {
                    return (
                      <div>
                        <label htmlFor={field.name}>Descripción:</label>
                        <input
                          id={field.name}
                          name={field.name}
                          value={field.state.value}
                          onBlur={field.handleBlur}
                          onChange={(e) => field.handleChange(e.target.value)}
                        />
                        <FieldInfo field={field} />
                      </div>
                    )
                  }}
                />
              </div>
            ))}
      </div>
      <button
        type="button"
        onClick={() =>
          hobbiesField.pushValue({
            name: '',
            description: '',
            yearsOfExperience: 0,
          })
        }
      >
        Agregar pasatiempo
      </button>
    </div>
  )}
/>
```

## Botones de Reinicio

Cuando se utiliza `<button type="reset">` junto con `form.reset()` de TanStack Form, debe evitar el comportamiento de reinicio HTML predeterminado para evitar reinicios inesperados de elementos del formulario (especialmente elementos `<select>`) a sus valores HTML iniciales.
Use `event.preventDefault()` dentro del manejador `onClick` del botón para evitar el reinicio nativo del formulario.

Ejemplo:

```tsx
<button
  type="reset"
  onClick={(event) => {
    event.preventDefault()
    form.reset()
  }}
>
  Reiniciar
</button>
```

Alternativamente, puede usar `<button type="button">` para evitar el reinicio HTML nativo.

```tsx
<button
  type="button"
  onClick={() => {
    form.reset()
  }}
>
  Reiniciar
</button>
```

Estos son los conceptos básicos y la terminología utilizada en la biblioteca `@tanstack/react-form`. Comprender estos conceptos le ayudará a trabajar de manera más efectiva con la biblioteca y crear formularios complejos con facilidad.
