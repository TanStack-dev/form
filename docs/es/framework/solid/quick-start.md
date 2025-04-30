---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:47:04.306Z'
id: quick-start
title: Inicio rápido
---

# Inicio Rápido

Lo mínimo necesario para comenzar con TanStack Form es crear un formulario y agregar un campo. Tenga en cuenta que este ejemplo no incluye validación ni manejo de errores... por ahora.

```tsx
import { createForm } from '@tanstack/solid-form'

function App() {
  const form = createForm(() => ({
    defaultValues: {
      fullName: '',
    },
    onSubmit: async ({ value }) => {
      // Do something with form data
      console.log(value)
    },
  }))

  return (
    <div>
      <h1>Simple Form Example</h1>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          form.handleSubmit()
        }}
      >
        <div>
          <form.Field
            name="fullName"
            children={(field) => (
              <input
                name={field().name}
                value={field().state.value}
                onBlur={field().handleBlur}
                onInput={(e) => field().handleChange(e.target.value)}
              />
            )}
          />
        </div>
        <button type="submit">Submit</button>
      </form>
    </div>
  )
}
```

A partir de aquí, estará listo para explorar todas las demás características de TanStack Form.
