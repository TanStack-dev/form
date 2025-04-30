---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:52:00.664Z'
id: form-validation
title: Validación de formularios
---

En el núcleo de las funcionalidades de TanStack Form está el concepto de validación. TanStack Form hace que la validación sea altamente personalizable:

- Usted puede controlar cuándo realizar la validación (al cambiar, al ingresar, al perder el foco, al enviar...)
- Las reglas de validación pueden definirse a nivel de campo o a nivel de formulario
- La validación puede ser síncrona o asíncrona (por ejemplo, como resultado de una llamada API)

## ¿Cuándo se realiza la validación?

¡Depende de usted! El componente `<form.Field />` acepta algunas funciones de retorno (callbacks) como propiedades, como `onChange` u `onBlur`. Estas funciones reciben el valor actual del campo, así como el objeto `fieldAPI`, para que pueda realizar la validación. Si encuentra un error de validación, simplemente devuelva el mensaje de error como cadena y estará disponible en `field.state.meta.errors`.

Aquí hay un ejemplo:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

En el ejemplo anterior, la validación se realiza en cada pulsación de tecla (`onchange`). Si, en cambio, quisiéramos que la validación se realice cuando el campo pierde el foco, cambiaríamos el código de la siguiente manera:

```svelte
<form.Field
  name="age"
  validators={{
    onBlur: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

Así que usted puede controlar cuándo se realiza la validación implementando la función de retorno deseada. Incluso puede realizar diferentes validaciones en diferentes momentos:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
    onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

En el ejemplo anterior, estamos validando diferentes aspectos del mismo campo en diferentes momentos (en cada pulsación de tecla y al perder el foco del campo). Dado que `field.state.meta.errors` es un arreglo, todos los errores relevantes en un momento dado se muestran. También puede usar `field.state.meta.errorMap` para obtener errores basados en _cuándo_ se realizó la validación (onChange, onBlur, etc.). Más información sobre cómo mostrar errores a continuación.

## Mostrar errores

Una vez que tiene configurada su validación, puede mapear los errores desde un arreglo para mostrarlos en su interfaz de usuario:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

O usar la propiedad `errorMap` para acceder al error específico que está buscando:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    {#if field.state.meta.errorMap['onChange']}
      <em role="alert">{field.state.meta.errorMap['onChange']}</em>
    {/if}
  {/snippet}
</form.Field>
```

Vale la pena mencionar que nuestro arreglo `errors` y el `errorMap` coinciden con los tipos devueltos por los validadores. Esto significa que:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    <!-- errorMap.onChange es de tipo `{isOldEnough: false} | undefined` -->
    <!-- meta.errors es de tipo `Array<{isOldEnough: false} | undefined>` -->
    {#if field.state.meta.errorMap['onChange']?.isOldEnough}
        <em>The user is not old enough</em>
    {/if}
  {/snippet}
</form.Field>
```

## Validación a nivel de campo vs a nivel de formulario

Como se mostró anteriormente, cada `<form.Field>` acepta sus propias reglas de validación a través de las funciones de retorno `onChange`, `onBlur`, etc. También es posible definir reglas de validación a nivel de formulario (en lugar de campo por campo) pasando funciones de retorno similares al hook `createForm()`.

Ejemplo:

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      age: 0,
    },
    onSubmit: async ({ value }) => {
      console.log(value)
    },
    validators: {
      // Agregue validadores al formulario de la misma manera que los agregaría a un campo
      onChange({ value }) {
        if (value.age < 13) {
          return 'Must be 13 or older to sign'
        }
        return undefined
      },
    },
  }))

  // Suscríbase al mapa de errores del formulario para que las actualizaciones se rendericen
  // alternativamente, puede usar `form.Subscribe`
  const formErrorMap = form.useStore((state) => state.errorMap)
</script>

<div>
  <!-- ... -->
  {#if formErrorMap.current.onChange}
    <div>
      <em>There was an error on the form: {formErrorMap.current.onChange}</em>
    </div>
  {/if}
  <!-- ... -->
</div>
```

## Validación funcional asíncrona

Si bien sospechamos que la mayoría de las validaciones serán síncronas, hay muchos casos en los que sería útil una llamada de red u otra operación asíncrona para validar.

Para esto, tenemos métodos dedicados como `onChangeAsync`, `onBlurAsync` y otros que se pueden usar para validar:

```svelte
<form.Field
  name="age"
  validators={{
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value < 13 ? 'You must be 13 to make an account' : undefined
    },
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

Las validaciones síncronas y asíncronas pueden coexistir. Por ejemplo, es posible definir tanto `onBlur` como `onBlurAsync` en el mismo campo:

```svelte
<form.Field
  name="age"
  validators={{
    onBlur: ({ value }) => (value < 13 ? 'You must be at least 13' : undefined),
    onBlurAsync: async ({ value }) => {
      const currentAge = await fetchCurrentAgeOnProfile()
      return value < currentAge ? 'You can only increase the age' : undefined
    },
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

El método de validación síncrono (`onBlur`) se ejecuta primero y el método asíncrono (`onBlurAsync`) solo se ejecuta si el síncrono (`onBlur`) tiene éxito. Para cambiar este comportamiento, establezca la opción `asyncAlways` en `true`, y el método asíncrono se ejecutará independientemente del resultado del método síncrono.

### Debounce integrado

Si bien las llamadas asíncronas son la mejor opción al validar contra una base de datos, ejecutar una solicitud de red en cada pulsación de tecla es una buena manera de saturar su base de datos.

En su lugar, habilitamos un método sencillo para aplicar debounce a sus llamadas `async` agregando una sola propiedad:

```svelte
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsync: async ({ value }) => {
      // ...
    },
  }}
>
  <!-- ... -->
</form.Field>
```

Esto aplicará un debounce a cada llamada asíncrona con un retraso de 500ms. Incluso puede anular esta propiedad por cada validación:

```svelte
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsyncDebounceMs: 1500,
    onChangeAsync: async ({ value }) => {
      // ...
    },
    onBlurAsync: async ({ value }) => {
      // ...
    },
  }}
>
  <!-- ... -->
</form.Field>
```

> Esto ejecutará `onChangeAsync` cada 1500ms mientras que `onBlurAsync` se ejecutará cada 500ms.

## Validación mediante bibliotecas de esquemas

Si bien las funciones proporcionan más flexibilidad y personalización sobre su validación, pueden ser un poco verbosas. Para ayudar a resolver esto, existen bibliotecas que proporcionan validación basada en esquemas para hacer que la validación abreviada y estricta en tipos sea sustancialmente más fácil. También puede definir un único esquema para todo su formulario y pasarlo a nivel de formulario, los errores se propagarán automáticamente a los campos.

### Bibliotecas de esquemas estándar

TanStack Form admite de forma nativa todas las bibliotecas que siguen la [especificación Standard Schema](https://github.com/standard-schema/standard-schema), más notablemente:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_Nota:_ asegúrese de usar la última versión de las bibliotecas de esquemas, ya que versiones anteriores podrían no admitir Standard Schema todavía.

Para usar esquemas de estas bibliotecas, puede pasarlos a las propiedades `validators` como lo haría con una función personalizada:

```svelte
<script>
  import { z } from 'zod'

  // ...

  const form = createForm(() => ({
    // ...
  }))
</script>

<form.Field
  name="age"
  validators={{
    onChange: z.number().gte(13, 'You must be 13 to make an account'),
  }}
>
  <!-- ... -->
</form.Field>
```

Las validaciones asíncronas a nivel de formulario y campo también son compatibles:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: z.number().gte(13, 'You must be 13 to make an account'),
    onChangeAsyncDebounceMs: 500,
    onChangeAsync: z.number().refine(
      async (value) => {
        const currentAge = await fetchCurrentAgeOnProfile()
        return value >= currentAge
      },
      {
        message: 'You can only increase the age',
      },
    ),
  }}
>
  <!-- ... -->
</form.Field>
```

## Evitar que se envíen formularios inválidos

Las funciones de retorno `onChange`, `onBlur`, etc., también se ejecutan cuando se envía el formulario y se bloquea el envío si el formulario es inválido.

El objeto de estado del formulario tiene una bandera `canSubmit` que es falsa cuando cualquier campo es inválido y el formulario ha sido modificado (`canSubmit` es verdadero hasta que el formulario ha sido modificado, incluso si algunos campos son "técnicamente" inválidos según sus propiedades `onChange`/`onBlur`).

Puede suscribirse a ella a través de `form.Subscribe` y usar el valor para, por ejemplo, deshabilitar el botón de envío cuando el formulario es inválido (en la práctica, los botones deshabilitados no son accesibles, use `aria-disabled` en su lugar).

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    /* ... */
  }))
</script>

<!-- ... -->

<!-- Botón de envío dinámico -->
<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
  children={(state) => (
    <button type="submit" disabled={!state().canSubmit}>
      {state().isSubmitting ? '...' : 'Submit'}
    </button>
  )}
>
  {#snippet children(state)}
    <button type="submit" disabled={!state.canSubmit}>
      {state.isSubmitting ? '...' : 'Submit'}
    </button>
  {/snippet}
</form.Subscribe>
```
