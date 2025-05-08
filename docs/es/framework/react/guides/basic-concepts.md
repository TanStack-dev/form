---
source-updated-at: '2025-05-08T07:42:29.000Z'
translation-updated-at: '2025-05-08T23:47:23.997Z'
id: basic-concepts
title: Conceptos básicos
---

# Conceptos Básicos y Terminología

Esta página introduce los conceptos básicos y la terminología utilizada en la biblioteca `@tanstack/react-form`. Familiarizarse con estos conceptos le ayudará a comprender y trabajar mejor con la biblioteca.

## Opciones del Formulario

Puede crear opciones para su formulario para que pueda ser compartido entre múltiples formularios utilizando la función `formOptions`.

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

Hay tres estados en los metadatos que pueden ser útiles para ver cómo interactúa el usuario con un campo:

- _"isTouched"_, después de que el usuario hace clic o tabula en el campo
- _"isPristine"_, hasta que el usuario cambia el valor del campo
- _"isDirty"_, después de que el valor del campo ha sido cambiado

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Estados del campo](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## Entendiendo 'isDirty' en Diferentes Bibliotecas

Estado `dirty` no persistente

- **Bibliotecas**: React Hook Form (RHF), Formik, Final Form.
- **Comportamiento**: Un campo está 'dirty' si su valor difiere del predeterminado. Revertir al valor predeterminado lo vuelve 'clean' nuevamente.

Estado `dirty` persistente

- **Bibliotecas**: Angular Form, Vue FormKit.
- **Comportamiento**: Un campo permanece 'dirty' una vez cambiado, incluso si se revierte al valor predeterminado.

Hemos elegido el modelo de estado 'dirty' persistente. Para también admitir un estado 'dirty' no persistente, introducimos la bandera `isDefault`. Esta bandera actúa como un inverso del estado 'dirty' no persistente.

```tsx
const { isTouched, isPristine, isDirty, isDefaultValue } = field.state.meta

// La siguiente línea recreará la funcionalidad `dirty` no persistente.
const nonPersistentIsDirty = !isDefaultValue
```

![Estados del campo extendidos](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states-extended.png)

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

`@tanstack/react-form` proporciona validación sincrónica y asincrónica de forma predeterminada. Las funciones de validación se pueden pasar al componente `form.Field` utilizando la propiedad `validators`.

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

Puede definir un esquema utilizando cualquiera de las bibliotecas que implementen la especificación y pasarlo a un validador de formulario o campo.

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

`@tanstack/react-form` ofrece varias formas de suscribirse a cambios de estado del formulario y campos, más notablemente el hook `useStore(form.store)` y el componente `form.Subscribe`. Estos métodos le permiten optimizar el rendimiento de renderizado de su formulario actualizando solo los componentes cuando sea necesario.

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

Puede encontrar más información en [Oyentes (Listeners)](./listeners.md)

## Campos de Arreglo (Array Fields)

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

## Botones de Reinicio (Reset Buttons)

Cuando se utiliza `<button type="reset">` junto con `form.reset()` de TanStack Form, debe prevenir el comportamiento de reinicio HTML predeterminado para evitar reinicios inesperados de elementos del formulario (especialmente elementos `<select>`) a sus valores HTML iniciales.
Use `event.preventDefault()` dentro del manejador `onClick` del botón para prevenir el reinicio nativo del formulario.

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

Alternativamente, puede usar `<button type="button">` para prevenir el reinicio HTML nativo.

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

Estos son los conceptos básicos y la terminología utilizados en la biblioteca `@tanstack/react-form`. Comprender estos conceptos le ayudará a trabajar de manera más efectiva con la biblioteca y crear formularios complejos con facilidad.
