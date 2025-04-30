---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:49:30.851Z'
id: basic-concepts
title: Conceptos básicos
---

# Conceptos Básicos y Terminología

Esta página introduce los conceptos básicos y la terminología utilizada en la biblioteca `@tanstack/vue-form`. Familiarizarse con estos conceptos le ayudará a comprender y trabajar mejor con la biblioteca.

## Opciones del Formulario

Puede crear opciones para su formulario de modo que puedan compartirse entre múltiples formularios utilizando la función `formOptions`.

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

Una Instancia del Formulario es un objeto que representa un formulario individual y proporciona métodos y propiedades para trabajar con él. Puede crear una instancia del formulario utilizando la función `useForm`. Esta función acepta un objeto con una función `onSubmit`, que se llama cuando se envía el formulario.

```js
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // Hacer algo con los datos del formulario
    console.log(value)
  },
})
```

También puede crear una instancia del formulario sin usar `formOptions` mediante la API independiente `useForm`:

```ts
const form = useForm({
  onSubmit: async ({ value }) => {
    // Hacer algo con los datos del formulario
    console.log(value)
  },
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## Campo

Un Campo representa un único elemento de entrada del formulario, como un campo de texto o una casilla de verificación. Los campos se crean utilizando el componente `form.Field` proporcionado por la instancia del formulario. El componente acepta una propiedad `name`, que debe coincidir con una clave en los valores predeterminados del formulario. También acepta un slot con ámbito definido por la directiva `v-slot`, que toma un objeto `field` como argumento.

Ejemplo:

```vue
<template>
  <!-- ... -->
  <form.Field name="fullName">
    <template v-slot="{ field }">
      <input
        :name="field.name"
        :value="field.state.value"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange(e.target.value)"
      />
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## Estado del Campo

Cada campo tiene su propio estado, que incluye su valor actual, estado de validación, mensajes de error y otros metadatos. Puede acceder al estado de un campo utilizando la propiedad `field.state`.

Ejemplo:

```js
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Hay tres estados del campo que pueden ser muy útiles para ver cómo interactúa el usuario con un campo. Un campo está _"touched"_ (tocado) cuando el usuario hace clic o se desplaza hasta él, _"pristine"_ (prístino) hasta que el usuario cambia su valor, y _"dirty"_ (modificado) después de que el valor ha sido cambiado. Puede verificar estos estados mediante las banderas `isTouched`, `isPristine` e `isDirty`, como se muestra a continuación.

```js
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Estados del campo](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## API del Campo

La API del Campo es un objeto proporcionado por un slot con ámbito utilizando la directiva `v-slot`. Este slot recibe un argumento llamado `field` que proporciona métodos y propiedades para trabajar con el estado del campo.

Ejemplo:

```vue
<template v-slot="{ field }">
  <input
    :name="field.name"
    :value="field.state.value"
    @blur="field.handleBlur"
    @input="(e) => field.handleChange(e.target.value)"
  />
</template>
```

## Validación

`@tanstack/vue-form` proporciona validación tanto síncrona como asíncrona de forma predeterminada. Las funciones de validación pueden pasarse al componente `form.Field` utilizando la propiedad `validators`.

Ejemplo:

```vue
<template>
    <!-- ... -->
    <form.Field
        name="firstName"
        :validators="{
            onChange: ({ value }) =>
                !value
                    ? `Se requiere un nombre`
                    : value.length < 3
                        ? `El nombre debe tener al menos 3 caracteres`
                        : undefined,
            onChangeAsync: async ({ value }) => {
                await new Promise((resolve) => setTimeout(resolve, 1000))
                return value.includes('error') && 'No se permite "error" en el nombre'
        },
    }"
    >
        <template v-slot="{ field }">
            <input
                :value="field.state.value"
                @input="(e) => field.handleChange(e.target.value)"
                @blur="field.handleBlur"
            />
            <FieldInfo :field="field" />
        </template>
    </form.Field>
    <!-- ... -->
</template>
```

## Validación con Bibliotecas de Esquema Estándar

Además de las opciones de validación personalizadas, también se admite la especificación [Standard Schema](https://github.com/standard-schema/standard-schema).

Puede definir un esquema utilizando cualquiera de las bibliotecas que implementen la especificación y pasarlo a un validador de formulario o campo.

Las bibliotecas admitidas incluyen:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'
import { z } from 'zod'

const form = useForm({
  // ...
})

const onChangeFirstName = z.string().refine(
  async (value) => {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return !value.includes('error')
  },
  {
    message: "No se permite 'error' en el nombre",
  },
)
</script>

<template>
  <!-- ... -->
  <form.Field
    name="firstName"
    :validators="{
      onChange: z.string().min(3, 'El nombre debe tener al menos 3 caracteres'),
      onChangeAsyncDebounceMs: 500,
      onChangeAsync: onChangeFirstName,
    }"
  >
    <template v-slot="{ field, state }">
      <label :htmlFor="field.name">Nombre:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).value)"
        @blur="field.handleBlur"
      />
      <FieldInfo :state="state" />
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## Reactividad

`@tanstack/vue-form` ofrece varias formas de suscribirse a cambios en el estado del formulario y los campos, destacando especialmente el método `form.useStore` y el componente `form.Subscribe`. Estos métodos le permiten optimizar el rendimiento de renderizado de su formulario actualizando solo los componentes cuando sea necesario.

Ejemplo:

```vue
<script setup lang="ts">
// ...
const firstName = form.useStore((state) => state.values.firstName)
</script>

<template>
  <!-- ... -->
  <form.Subscribe>
    <template v-slot="{ canSubmit, isSubmitting }">
      <button type="submit" :disabled="!canSubmit">
        {{ isSubmitting ? '...' : 'Enviar' }}
      </button>
    </template>
  </form.Subscribe>
  <!-- ... -->
</template>
```

## Oyentes (Listeners)

`@tanstack/vue-form` le permite reaccionar a desencadenantes específicos y "escucharlos" para ejecutar efectos secundarios.

Ejemplo:

```vue
<template>
  <form.Field
    name="country"
    :listeners="{
      onChange: ({ value }) => {
        console.log(`País cambiado a: ${value}, reiniciando provincia`)
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
</template>
```

Puede encontrar más información en [Oyentes (Listeners)](./listeners.md).

Nota: Se desaconseja el uso del método `form.useField` para lograr reactividad, ya que está diseñado para usarse cuidadosamente dentro del componente `form.Field`. En su lugar, puede usar `form.useStore`.

## Campos de Arreglo

Los campos de arreglo le permiten administrar una lista de valores dentro de un formulario, como una lista de pasatiempos. Puede crear un campo de arreglo utilizando el componente `form.Field` con la propiedad `mode="array"`.

Al trabajar con campos de arreglo, puede usar los métodos `pushValue`, `removeValue`, `swapValues` y `moveValue` para agregar, eliminar e intercambiar valores en el arreglo.

Ejemplo:

```vue
<template>
  <form @submit.prevent.stop="form.handleSubmit">
    <form.Field name="hobbies" mode="array">
      <template v-slot="{ field: hobbiesField }">
        <div>
          Pasatiempos
          <div>
            <div
              v-if="
                Array.isArray(hobbiesField.state.value) &&
                !hobbiesField.state.value.length
              "
            >
              No se encontraron pasatiempos.
            </div>
            <div v-else>
              <div v-for="(_, i) in hobbiesField.state.value" :key="i">
                <form.Field :name="`hobbies[${i}].name`">
                  <template v-slot="{ field }">
                    <div>
                      <label :for="field.name">Nombre:</label>
                      <input
                        :id="field.name"
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="(e) => field.handleChange(e.target.value)"
                      />
                      <button
                        type="button"
                        @click="hobbiesField.removeValue(i)"
                      >
                        X
                      </button>
                      <FieldInfo :field="field" />
                    </div>
                  </template>
                </form.Field>
                <form.Field :name="`hobbies[${i}].description`">
                  <template v-slot="{ field }">
                    <div>
                      <label :for="field.name">Descripción:</label>
                      <input
                        :id="field.name"
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="(e) => field.handleChange(e.target.value)"
                      />
                      <FieldInfo :field="field" />
                    </div>
                  </template>
                </form.Field>
              </div>
            </div>
            <button
              type="button"
              @click="
                hobbiesField.pushValue({
                  name: '',
                  description: '',
                  yearsOfExperience: 0,
                })
              "
            >
              Agregar pasatiempo
            </button>
          </div>
        </div>
      </template>
    </form.Field>
  </form>
</template>
```

Estos son los conceptos básicos y la terminología utilizados en la biblioteca `@tanstack/vue-form`. Comprender estos conceptos le ayudará a trabajar de manera más efectiva con la biblioteca y crear formularios complejos con facilidad.
