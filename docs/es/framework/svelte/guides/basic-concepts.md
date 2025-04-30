---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:51:12.760Z'
id: basic-concepts
title: Conceptos básicos
---

# Conceptos Básicos y Terminología

Esta página introduce los conceptos básicos y la terminología utilizada en la biblioteca `@tanstack/svelte-form`. Familiarizarse con estos conceptos le ayudará a comprender y trabajar mejor con la biblioteca.

## Opciones del Formulario

Puede crear opciones para su formulario para que puedan compartirse entre múltiples formularios utilizando la función `formOptions`.

Ejemplo:

```ts
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## Instancia del Formulario

Una Instancia del Formulario es un objeto que representa un formulario individual y proporciona métodos y propiedades para trabajar con el formulario. Se crea una instancia del formulario utilizando la función `createForm`. La función acepta un objeto con una función `onSubmit`, que se llama cuando se envía el formulario.

```ts
const form = createForm(() => ({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // Hacer algo con los datos del formulario
    console.log(value)
  },
}))
```

También puede crear una instancia del formulario sin usar `formOptions`:

```ts
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

Un Campo representa un único elemento de entrada de formulario, como un campo de texto o una casilla de verificación. Los campos se crean utilizando el componente `form.Field` proporcionado por la instancia del formulario. El componente acepta una propiedad `name`, que debe coincidir con una clave en los valores predeterminados del formulario. También acepta una propiedad `children`, que es una función de renderizado que toma un objeto de campo como argumento.

Ejemplo:

```svelte
<form.Field name="firstName">
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onblur={field.handleBlur}
      oninput={(e) => field.handleChange(e.target.value)}
    />
  {/snippet}
</form.Field>
```

## Estado del Campo

Cada campo tiene su propio estado, que incluye su valor actual, estado de validación, mensajes de error y otros metadatos. Puede acceder al estado de un campo utilizando la propiedad `field.state`.

Ejemplo:

```ts
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Hay tres estados del campo que pueden ser muy útiles para ver cómo interactúa el usuario con un campo. Un campo está _"touched"_ (tocado) cuando el usuario hace clic o tabula en él, _"pristine"_ (prístino) hasta que el usuario cambia su valor, y _"dirty"_ (modificado) después de que el valor ha sido cambiado. Puede verificar estos estados mediante las banderas `isTouched`, `isPristine` e `isDirty`, como se muestra a continuación.

```ts
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Estados del campo](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## API del Campo

La API del Campo es un objeto pasado a la función de renderizado cuando se crea un campo. Proporciona métodos para trabajar con el estado del campo.

Ejemplo:

```svelte
<input
  name={field.name}
  value={field.state.value}
  onblur={field.handleBlur}
  oninput={(e) => field.handleChange(e.target.value)}
/>
```

## Validación

`@tanstack/svelte-form` proporciona validación tanto síncrona como asíncrona de forma predeterminada. Las funciones de validación se pueden pasar al componente `form.Field` utilizando la propiedad `validators`.

Ejemplo:

```svelte
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
>
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onBlur={field.handleBlur}
      onInput={(e) => field.handleChange(e.target.value)}
    />
    <p>{field.state.meta.errors[0]}</p>
  {/snippet}
</form.Field>
```

## Validación con Bibliotecas de Esquema Estándar

Además de las opciones de validación personalizadas, también admitimos la especificación [Standard Schema](https://github.com/standard-schema/standard-schema).

Puede definir un esquema utilizando cualquiera de las bibliotecas que implementen la especificación y pasarlo a un validador de formulario o campo.

Las bibliotecas admitidas incluyen:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```svelte
<script>
  import { z } from 'zod'

  // ...
</script>

<form.Field
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
>
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onBlur={field.handleBlur}
      onInput={(e) => field.handleChange(e.target.value)}
    />
    <p>{field.state.meta.errors[0]}</p>
  {/snippet}
</form.Field>
```

## Reactividad

`@tanstack/svelte-form` ofrece varias formas de suscribirse a cambios en el estado del formulario y los campos, destacando especialmente el hook `form.useStore` y el componente `form.Subscribe`. Estos métodos le permiten optimizar el rendimiento de renderizado de su formulario actualizando solo los componentes cuando sea necesario.

Ejemplo:

```svelte
<script>
  //...
  const firstName = form.useStore((state) => state.values.firstName)
</script>

<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
>
  {#snippet children(state)}
    <button type="submit" disabled={!state.canSubmit}>
      {state.isSubmitting ? '...' : 'Enviar'}
    </button>
  {/snippet}
</form.Subscribe>
```

## Campos de Arreglo

Los campos de arreglo le permiten administrar una lista de valores dentro de un formulario, como una lista de pasatiempos. Puede crear un campo de arreglo utilizando el componente `form.Field` con la propiedad `mode="array"`.

Cuando trabaje con campos de arreglo, puede usar los métodos `pushValue`, `removeValue`, `swapValues` y `moveValue` para agregar, eliminar e intercambiar valores en el arreglo.

Ejemplo:

```svelte
<form.Field name="hobbies" mode="array">
  {#snippet children(hobbiesField)}
    <div>
      Pasatiempos
      <div>
        {#each hobbiesField.state.value as _, i}
            <div>
              <form.Field name={`hobbies[${i}].name`}>
                {#snippet children(field)}
                  <div>
                    <label for={field.name}>Nombre:</label>
                    <input
                      id={field.name}
                      name={field.name}
                      value={field.state.value}
                      onblur={field.handleBlur}
                      onchange={(e) => field.handleChange(e.target.value)}
                    />
                    <button
                      type="button"
                      onclick={() => hobbiesField.removeValue(i)}
                    >
                      X
                    </button>
                  </div>
                {/snippet}
              </form.Field>
              <form.Field name={`hobbies[${i}].description`}>
                {#snippet children(field)}
                    <div>
                      <label for={field.name}>Descripción:</label>
                      <input
                        id={field.name}
                        name={field.name}
                        value={field.state.value}
                        onblur={field.handleBlur}
                        onchange={(e) => field.handleChange(e.target.value)}
                      />
                    </div>
                {/snippet}
              </form.Field>
            </div>
          {:else}
            No se encontraron pasatiempos.
          {/each}
      </div>
      <button
        type="button"
        onclick={() =>
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
  {/snippet}
</form.Field>
```

Estos son los conceptos básicos y la terminología utilizados en la biblioteca `@tanstack/svelte-form`. Comprender estos conceptos le ayudará a trabajar de manera más efectiva con la biblioteca y crear formularios complejos con facilidad.
