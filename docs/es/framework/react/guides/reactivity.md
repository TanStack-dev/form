---
source-updated-at: '2025-03-20T03:01:05.000Z'
translation-updated-at: '2025-04-30T21:47:52.413Z'
id: reactivity
title: Reactividad
---

# Reactividad

TanStack Form no provoca re-renderizaciones al interactuar con el formulario. Por lo tanto, puede que intentes usar un valor del estado del formulario o de un campo sin éxito.

Si deseas acceder a valores reactivos, necesitarás suscribirte a ellos utilizando uno de estos dos métodos: `useStore` o el componente `form.Subscribe`.

Algunos usos para estas suscripciones son renderizar valores actualizados de los campos, determinar qué renderizar basado en una condición o usar valores de los campos dentro de la lógica de tu componente.

> Para situaciones donde quieras "reaccionar" a desencadenadores, revisa la API de [listeners](./listeners.md).

## useStore

El hook `useStore` es ideal cuando necesitas acceder a valores del formulario dentro de la lógica de tu componente. `useStore` recibe dos parámetros. Primero, el store del formulario. Segundo, un selector para afinar la parte del formulario a la que deseas suscribirte.

```tsx
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
```

Puedes acceder a cualquier parte del estado del formulario en el selector.

> Ten en cuenta que `useStore` provocará una re-renderización completa del componente cada vez que cambie el valor al que estás suscrito.

Si bien ES posible omitir el selector, resiste la tentación, ya que omitirlo resultaría en muchas re-renderizaciones innecesarias cada vez que cambie cualquier parte del estado del formulario.

## form.Subscribe

El componente `form.Subscribe` es más adecuado cuando necesitas reaccionar a algo dentro de la interfaz de usuario de tu componente. Por ejemplo, mostrar u ocultar elementos basados en el valor de un campo del formulario.

```tsx
<form.Subscribe
  selector={(state) => state.values.firstName}
  children={(firstName) => (
    <form.Field>
      {(field) => (
        <input
          name="lastName"
          value={field.state.lastName}
          onChange={field.handleChange}
        />
      )}
    </form.Field>
  )}
/>
```

> El componente `form.Subscribe` no desencadena re-renderizaciones a nivel de componente. Cada vez que cambia el valor al que estás suscrito, solo el componente `form.Subscribe` se re-renderiza.

La elección entre usar `useStore` o `form.Subscribe` se reduce principalmente a lo siguiente: si se renderiza en la interfaz de usuario, opta por `form.Subscribe` por sus ventajas de optimización, y si necesitas reactividad dentro de la lógica, entonces `useStore` es la opción a elegir.
