---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T21:51:10.741Z'
id: form-validation
title: Validación de formularios
---

En el núcleo de las funcionalidades de TanStack Form se encuentra el concepto de validación. TanStack Form hace que la validación sea altamente personalizable:

- Puede controlar cuándo realizar la validación (al cambiar, al ingresar, al perder el foco, al enviar...)
- Las reglas de validación pueden definirse a nivel de campo o a nivel de formulario
- La validación puede ser síncrona o asíncrona (por ejemplo, como resultado de una llamada API)

## ¿Cuándo se realiza la validación?

¡Depende de usted! El componente `<Field />` acepta algunas funciones de retorno (callbacks) como propiedades, como `onChange` u `onBlur`. Estas funciones reciben el valor actual del campo, así como el objeto fieldAPI, para que pueda realizar la validación. Si encuentra un error de validación, simplemente devuelva el mensaje de error como cadena y estará disponible en `field().state.meta.errors`.

Aquí hay un ejemplo:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => (
    <>
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {!field().state.meta.isValid ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

En el ejemplo anterior, la validación se realiza en cada pulsación de tecla (`onChange`). Si, en cambio, quisiéramos que la validación se realice cuando el campo pierde el foco, cambiaríamos el código anterior de la siguiente manera:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => (
    <>
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        // Escuchar el evento onBlur en el campo
        onBlur={field().handleBlur}
        // Siempre necesitamos implementar onInput para que TanStack Form reciba los cambios
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {!field().state.meta.isValid ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

Así que puede controlar cuándo se realiza la validación implementando la función de retorno deseada. Incluso puede realizar diferentes validaciones en diferentes momentos:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
    onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
  }}
>
  {(field) => (
    <>
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        // Escuchar el evento onBlur en el campo
        onBlur={field().handleBlur}
        // Siempre necesitamos implementar onInput para que TanStack Form reciba los cambios
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {!field().state.meta.isValid ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

En el ejemplo anterior, estamos validando diferentes aspectos del mismo campo en diferentes momentos (en cada pulsación de tecla y al perder el foco del campo). Dado que `field().state.meta.errors` es un arreglo, se muestran todos los errores relevantes en un momento dado. También puede usar `field().state.meta.errorMap` para obtener errores basados en _cuándo_ se realizó la validación (onChange, onBlur, etc.). Más información sobre cómo mostrar errores a continuación.

## Mostrar errores

Una vez que tenga su validación implementada, puede mapear los errores desde un arreglo para mostrarlos en su interfaz de usuario:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => {
    return (
      <>
        {/* ... */}
        {!field().state.meta.isValid ? (
          <em>{field().state.meta.errors.join(',')}</em>
        ) : null}
      </>
    )
  }}
</form.Field>
```

O usar la propiedad `errorMap` para acceder al error específico que está buscando:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => (
    <>
      {/* ... */}
      {field().state.meta.errorMap['onChange'] ? (
        <em>{field().state.meta.errorMap['onChange']}</em>
      ) : null}
    </>
  )}
</form.Field>
```

Vale la pena mencionar que nuestro arreglo `errors` y el `errorMap` coinciden con los tipos devueltos por los validadores. Esto significa que:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }}
>
  {(field) => (
    <>
      {/* ... */}
      {/* errorMap.onChange es de tipo `{isOldEnough: false} | undefined` */}
      {/* meta.errors es de tipo `Array<{isOldEnough: false} | undefined>` */}
      {!field().state.meta.errorMap['onChange']?.isOldEnough ? (
        <em>The user is not old enough</em>
      ) : null}
    </>
  )}
</form.Field>
```

## Validación a nivel de campo vs a nivel de formulario

Como se mostró anteriormente, cada `<Field>` acepta sus propias reglas de validación a través de las funciones de retorno `onChange`, `onBlur`, etc. También es posible definir reglas de validación a nivel de formulario (en lugar de campo por campo) pasando funciones de retorno similares al hook `createForm()`.

Ejemplo:

```tsx
export default function App() {
  const form = createForm(() => ({
    defaultValues: {
      age: 0,
    },
    onSubmit: async ({ value }) => {
      console.log(value)
    },
    validators: {
      // Agregar validadores al formulario de la misma manera que los agregaría a un campo
      onChange({ value }) {
        if (value.age < 13) {
          return 'Must be 13 or older to sign'
        }
        return undefined
      },
    },
  }))

  // Suscribirse al mapa de errores del formulario para que las actualizaciones se rendericen
  // alternativamente, puede usar `form.Subscribe`
  const formErrorMap = form.useStore((state) => state.errorMap)

  return (
    <div>
      {/* ... */}
      {formErrorMap().onChange ? (
        <div>
          <em>There was an error on the form: {formErrorMap().onChange}</em>
        </div>
      ) : null}
      {/* ... */}
    </div>
  )
}
```

### Establecer errores a nivel de campo desde los validadores del formulario

Puede establecer errores en los campos desde los validadores del formulario. Un caso de uso común para esto es validar todos los campos al enviar llamando a un único punto final API en el validador `onSubmitAsync` del formulario.

```tsx
import { Show } from 'solid-js'
import { createForm } from '@tanstack/solid-form'

export default function App() {
  const form = createForm(() => ({
    defaultValues: {
      age: 0,
      socials: [],
      details: {
        email: '',
      },
    },
    validators: {
      onSubmitAsync: async ({ value }) => {
        // Validar el valor en el servidor
        const hasErrors = await validateDataOnServer(value)
        if (hasErrors) {
          return {
            form: 'Invalid data', // La clave `form` es opcional
            fields: {
              age: 'Must be 13 or older to sign',
              // Establecer errores en campos anidados con el nombre del campo
              'socials[0].url': 'The provided URL does not exist',
              'details.email': 'An email is required',
            },
          }
        }

        return null
      },
    },
  }))

  return (
    <div>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          void form.handleSubmit()
        }}
      >
        <form.Field
          name="age"
          children={(field) => (
            <>
              <label for={field().name}>Age:</label>
              <input
                id={field().name}
                name={field().name}
                value={field().state.value}
                type="number"
                onChange={(e) => field().handleChange(e.target.valueAsNumber)}
              />
              <Show when={field().state.meta.errors.length > 0}>
                <em role="alert">{field().state.meta.errors.join(', ')}</em>
              </Show>
            </>
          )}
        />
        <form.Subscribe
          selector={(state) => ({ errors: state.errors })}
          children={(state) => (
            <Show when={state().errors.length > 0}>
              <div>
                <em>
                  There was an error on the form: {state().errors.join(', ')}
                </em>
              </div>
            </Show>
          )}
        />

        <button type="submit">Submit</button>
        {/*...*/}
      </form>
    </div>
  )
}
```

> Algo que vale la pena mencionar es que si tiene una función de validación de formulario que devuelve un error, ese error puede sobrescribirse por la validación específica del campo.
>
> Esto significa que:
>
> ```tsx
>  const form = createForm(() => ({
>    defaultValues: {
>      age: 0,
>    },
>    validators: {
>      onChange: ({ value }) => {
>        return {
>          fields: {
>            age: value.age < 12 ? 'Too young!' : undefined,
>          },
>        };
>      },
>    },
>  }));
>
>  return (
>    <form.Field
>      name="age"
>      validators={{
>        onChange: ({ value }) => (value % 2 === 0 ? 'Must be odd!' : undefined),
>      }}
>      children={() => <>{/* ... */}</>}
>    />
>  );
> }
> ```
>
> Solo mostrará `'Must be odd!'` incluso si el error 'Too young!' es devuelto por la validación a nivel de formulario.

## Validación Funcional Asíncrona

Si bien sospechamos que la mayoría de las validaciones serán síncronas, hay muchas instancias donde una llamada de red u otra operación asíncrona sería útil para validar.

Para hacer esto, tenemos métodos dedicados `onChangeAsync`, `onBlurAsync` y otros que pueden usarse para validar:

```tsx
<form.Field
  name="age"
  validators={{
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value < 13 ? 'You must be 13 to make an account' : undefined
    },
  }}
>
  {(field) => (
    <>
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {!field().state.meta.isValid ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

Las validaciones síncronas y asíncronas pueden coexistir. Por ejemplo, es posible definir tanto `onBlur` como `onBlurAsync` en el mismo campo:

```tsx
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
  {(field) => (
    <>
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        onBlur={field().handleBlur}
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {field().state.meta.errors ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

El método de validación síncrono (`onBlur`) se ejecuta primero y el método asíncrono (`onBlurAsync`) solo se ejecuta si el síncrono (`onBlur`) tiene éxito. Para cambiar este comportamiento, establezca la opción `asyncAlways` en `true`, y el método asíncrono se ejecutará independientemente del resultado del método síncrono.

### Debouncing incorporado

Si bien las llamadas asíncronas son el camino a seguir cuando se valida contra la base de datos, ejecutar una solicitud de red en cada pulsación de tecla es una buena manera de hacer un DDoS a su base de datos.

En su lugar, habilitamos un método fácil para debouncear sus llamadas `async` agregando una sola propiedad:

```tsx
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsync: async ({ value }) => {
      // ...
    },
  }}
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

Esto debounceará cada llamada asíncrona con un retraso de 500ms. Incluso puede anular esta propiedad por cada propiedad de validación:

```tsx
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
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

Esto ejecutará `onChangeAsync` cada 1500ms mientras que `onBlurAsync` se ejecutará cada 500ms.

## Validación a través de Bibliotecas de Esquemas

Si bien las funciones proporcionan más flexibilidad y personalización sobre su validación, pueden ser un poco verbosas. Para ayudar a resolver esto, hay bibliotecas que proporcionan validación basada en esquemas para hacer la validación abreviada y estricta en tipos sustancialmente más fácil. También puede definir un único esquema para todo su formulario y pasarlo al nivel del formulario, los errores se propagarán automáticamente a los campos.

### Bibliotecas de Esquemas Estándar

TanStack Form admite de forma nativa todas las bibliotecas que siguen la [especificación Standard Schema](https://github.com/standard-schema/standard-schema), más notablemente:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_Nota:_ asegúrese de usar la última versión de las bibliotecas de esquemas, ya que versiones anteriores podrían no admitir Standard Schema todavía.

> La validación no le proporcionará valores transformados. Consulte [manejo de envío](./submission-handling.md) para más información.

Para usar esquemas de estas bibliotecas, puede pasarlos a las propiedades `validators` como lo haría con una función personalizada:

```tsx
import { z } from 'zod'

// ...

const form = createForm(() => ({
  // ...
}))

;<form.Field
  name="age"
  validators={{
    onChange: z.number().gte(13, 'You must be 13 to make an account'),
  }}
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

Las validaciones asíncronas a nivel de formulario y campo también son compatibles:

```tsx
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
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

Si necesita aún más control sobre su validación de Standard Schema, puede combinar un Standard Schema
