---
source-updated-at: '2025-03-14T21:54:05.000Z'
translation-updated-at: '2025-04-30T21:50:06.400Z'
id: basic-concepts
title: Conceptos básicos
---

# Conceptos Básicos y Terminología

Esta página introduce los conceptos básicos y la terminología utilizada en la biblioteca `@tanstack/solid-form`. Familiarizarse con estos conceptos le ayudará a comprender y trabajar mejor con la biblioteca.

## Opciones del Formulario

Puede crear opciones para su formulario para que puedan ser compartidas entre múltiples formularios utilizando la función `formOptions`.

Ejemplo:

```tsx
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## Instancia del Formulario

Una Instancia del Formulario es un objeto que representa un formulario individual y proporciona métodos y propiedades para trabajar con el formulario. Se crea una instancia del formulario utilizando el hook `createForm` proporcionado por las opciones del formulario. El hook acepta un objeto con una función `onSubmit`, que se llama cuando se envía el formulario.

```tsx
const form = createForm(() => ({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // Hacer algo con los datos del formulario
    console.log(value)
  },
}))
```

También puede crear una instancia del formulario sin usar `formOptions` utilizando la API independiente `createForm`:

```tsx
const form = createForm<Person>(() => ({
  onSubmit: async ({ value }) => {
    // Hacer algo con los datos del formulario
    console.log(value)
  },
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  },
}))
```

## Campo

Un Campo representa un único elemento de entrada del formulario, como un campo de texto o una casilla de verificación. Los campos se crean utilizando el componente `form.Field` proporcionado por la instancia del formulario. El componente acepta una prop `name`, que debe coincidir con una clave en los valores predeterminados del formulario. También acepta una prop `children`, que es una función de renderizado que toma un objeto de campo como argumento.

Ejemplo:

```tsx
<form.Field
  name="firstName"
  children={(field) => (
    <input
      name={field().name}
      value={field().state.value}
      onBlur={field().handleBlur}
      onInput={(e) => field().handleChange(e.target.value)}
    />
  )}
/>
```

## Estado del Campo

Cada campo tiene su propio estado, que incluye su valor actual, estado de validación, mensajes de error y otros metadatos. Puede acceder al estado de un campo utilizando la propiedad `field().state`.

Ejemplo:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field().state
```

Hay tres estados del campo que pueden ser muy útiles para ver cómo interactúa el usuario con un campo. Un campo está _"touched"_ (tocado) cuando el usuario hace clic o tabula en él, _"pristine"_ (prístino) hasta que el usuario cambia su valor, y _"dirty"_ (modificado) después de que se ha cambiado el valor. Puede verificar estos estados a través de las banderas `isTouched`, `isPristine` e `isDirty`, como se muestra a continuación.

```tsx
const { isTouched, isPristine, isDirty } = field().state.meta
```

![Estados del campo](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## API del Campo

La API del Campo es un objeto pasado a la función de renderizado cuando se crea un campo. Proporciona métodos para trabajar con el estado del campo.

Ejemplo:

```tsx
<input
  name={field().name}
  value={field().state.value}
  onBlur={field().handleBlur}
  onInput={(e) => field().handleChange(e.target.value)}
/>
```

## Validación

`@tanstack/solid-form` proporciona validación tanto síncrona como asíncrona de forma predeterminada. Las funciones de validación pueden pasarse al componente `form.Field` utilizando la prop `validators`.

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
        name={field().name}
        value={field().state.value}
        onBlur={field().handleBlur}
        onInput={(e) => field().handleChange(e.target.value)}
      />
      <p>{field().state.meta.errors[0]}</p>
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

// ...
;<form.Field
  name="firstName"
  validators={{
    onChange: z.string().min(3, 'El nombre debe tener al menos 3 caracteres'),
    onChangeAsyncDebounceMs: 500,
    onChangeAsync: z.string().refine(
      async (value) => {
        await new Promise((resolve) => setTimeout(resolve, 1000))
        return !value.includes('error')
      },
      {
        message: 'No se permite "error" en el nombre',
      },
    ),
  }}
  children={(field) => (
    <>
      <input
        name={field().name}
        value={field().state.value}
        onBlur={field().handleBlur}
        onInput={(e) => field().handleChange(e.target.value)}
      />
      <p>{field().state.meta.errors[0]}</p>
    </>
  )}
/>
```

## Reactividad

`@tanstack/solid-form` ofrece varias formas de suscribirse a los cambios de estado del formulario y los campos, destacando especialmente el hook `form.useStore` y el componente `form.Subscribe`. Estos métodos le permiten optimizar el rendimiento de renderizado de su formulario actualizando solo los componentes cuando sea necesario.

Ejemplo:

```tsx
const firstName = form.useStore((state) => state.values.firstName)
//...
<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
  children={(state) => (
    <button type="submit" disabled={!state().canSubmit}>
      {state().isSubmitting ? '...' : 'Enviar'}
    </button>
  )}
/>
```

## Campos de Arreglo

Los campos de arreglo le permiten administrar una lista de valores dentro de un formulario, como una lista de pasatiempos. Puede crear un campo de arreglo utilizando el componente `form.Field` con la prop `mode="array"`.

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
        <Show
          when={hobbiesField().state.value.length > 0}
          fallback={'No se encontraron pasatiempos.'}
        >
          <Index each={hobbiesField().state.value}>
            {(_, i) => (
              <div>
                <form.Field
                  name={`hobbies[${i}].name`}
                  children={(field) => (
                    <div>
                      <label for={field().name}>Nombre:</label>
                      <input
                        id={field().name}
                        name={field().name}
                        value={field().state.value}
                        onBlur={field().handleBlur}
                        onInput={(e) => field().handleChange(e.target.value)}
                      />
                      <button
                        type="button"
                        onClick={() => hobbiesField().removeValue(i)}
                      >
                        X
                      </button>
                    </div>
                  )}
                />
                <form.Field
                  name={`hobbies[${i}].description`}
                  children={(field) => {
                    return (
                      <div>
                        <label for={field().name}>Descripción:</label>
                        <input
                          id={field().name}
                          name={field().name}
                          value={field().state.value}
                          onBlur={field().handleBlur}
                          onInput={(e) => field().handleChange(e.target.value)}
                        />
                      </div>
                    )
                  }}
                />
              </div>
            )}
          </Index>
        </Show>
      </div>
      <button
        type="button"
        onClick={() =>
          hobbiesField().pushValue({
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

Estos son los conceptos básicos y la terminología utilizada en la biblioteca `@tanstack/solid-form`. Comprender estos conceptos le ayudará a trabajar de manera más efectiva con la biblioteca y crear formularios complejos con facilidad.
