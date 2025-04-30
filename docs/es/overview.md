---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T21:47:34.296Z'
id: overview
title: Resumen
---

# Visión general

TanStack Form es la solución definitiva para manejar formularios en aplicaciones web, proporcionando un enfoque potente y flexible para la gestión de formularios. Diseñado con soporte de primera clase para TypeScript, componentes de interfaz de usuario sin cabeza (headless UI components) y un diseño independiente del framework, simplifica el manejo de formularios y garantiza una experiencia perfecta en varios frameworks de front-end.

## Motivación

La mayoría de los frameworks web no ofrecen una solución integral para el manejo de formularios, lo que obliga a los desarrolladores a crear sus propias implementaciones personalizadas o depender de bibliotecas menos capaces. Esto a menudo resulta en falta de consistencia, bajo rendimiento y mayor tiempo de desarrollo. TanStack Form busca abordar estos desafíos proporcionando una solución todo en uno para gestionar formularios que es tanto potente como fácil de usar.

Con TanStack Form, los desarrolladores pueden abordar desafíos comunes relacionados con formularios, como:

- Vinculación de datos reactiva (reactive data binding) y gestión del estado (state management)
- Validación compleja y manejo de errores
- Accesibilidad y diseño responsivo (responsive design)
- Internacionalización (internationalization) y localización (localization)
- Compatibilidad multiplataforma (cross-platform compatibility) y estilos personalizados (custom styling)

Al proporcionar una solución completa para estos desafíos, TanStack Form permite a los desarrolladores construir formularios robustos y fáciles de usar con facilidad.

## Basta de hablar, ¡muéstrenme algo de código ya!

En el siguiente ejemplo, puede ver TanStack Form en acción con el adaptador para el framework React:

[Abrir en CodeSandbox](https://codesandbox.io/s/github/tanstack/form/tree/main/examples/react/simple)

```tsx
import * as React from 'react'
import { createRoot } from 'react-dom/client'
import { useForm } from '@tanstack/react-form'
import type { AnyFieldApi } from '@tanstack/react-form'

function FieldInfo({ field }: { field: AnyFieldApi }) {
  return (
    <>
      {field.state.meta.isTouched && !field.state.meta.isValid ? (
        <em>{field.state.meta.errors.join(', ')}</em>
      ) : null}
      {field.state.meta.isValidating ? 'Validating...' : null}
    </>
  )
}

export default function App() {
  const form = useForm({
    defaultValues: {
      firstName: '',
      lastName: '',
    },
    onSubmit: async ({ value }) => {
      // Do something with form data
      console.log(value)
    },
  })

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
          {/* A type-safe field component*/}
          <form.Field
            name="firstName"
            validators={{
              onChange: ({ value }) =>
                !value
                  ? 'A first name is required'
                  : value.length < 3
                    ? 'First name must be at least 3 characters'
                    : undefined,
              onChangeAsyncDebounceMs: 500,
              onChangeAsync: async ({ value }) => {
                await new Promise((resolve) => setTimeout(resolve, 1000))
                return (
                  value.includes('error') && 'No "error" allowed in first name'
                )
              },
            }}
            children={(field) => {
              // Avoid hasty abstractions. Render props are great!
              return (
                <>
                  <label htmlFor={field.name}>First Name:</label>
                  <input
                    id={field.name}
                    name={field.name}
                    value={field.state.value}
                    onBlur={field.handleBlur}
                    onChange={(e) => field.handleChange(e.target.value)}
                  />
                  <FieldInfo field={field} />
                </>
              )
            }}
          />
        </div>
        <div>
          <form.Field
            name="lastName"
            children={(field) => (
              <>
                <label htmlFor={field.name}>Last Name:</label>
                <input
                  id={field.name}
                  name={field.name}
                  value={field.state.value}
                  onBlur={field.handleBlur}
                  onChange={(e) => field.handleChange(e.target.value)}
                />
                <FieldInfo field={field} />
              </>
            )}
          />
        </div>
        <form.Subscribe
          selector={(state) => [state.canSubmit, state.isSubmitting]}
          children={([canSubmit, isSubmitting]) => (
            <button type="submit" disabled={!canSubmit}>
              {isSubmitting ? '...' : 'Submit'}
            </button>
          )}
        />
      </form>
    </div>
  )
}

const rootElement = document.getElementById('root')!

createRoot(rootElement).render(<App />)
```

## Me convencieron, ¿y ahora qué?

- Aprenda TanStack Form a su propio ritmo con nuestra completa [Guía paso a paso](../installation) y [Referencia de API](../reference/classes/formapi)
