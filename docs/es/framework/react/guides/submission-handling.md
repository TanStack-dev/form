---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-04-30T21:48:00.135Z'
id: submission-handling
title: Manejo de envíos
---

## Pasar datos adicionales al manejo de envío

Puede tener múltiples tipos de comportamiento de envío, por ejemplo, volver a otra página o permanecer en el formulario.  
Puede lograr esto especificando la propiedad `onSubmitMeta`. Estos metadatos se pasarán a la función `onSubmit`.

> Nota: si `form.handleSubmit()` se llama sin metadatos, utilizará los valores predeterminados proporcionados.

```tsx
import { useForm } from '@tanstack/react-form'

type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null
}

// Los metadatos no son obligatorios para llamar a form.handleSubmit().
// Especifique qué valores usar como predeterminados si no se pasan metadatos
const defaultMeta: FormMeta = {
  submitAction: null,
}

function App() {
  const form = useForm({
    defaultValues: {
      data: '',
    },
    // Defina qué valores de metadatos esperar en el envío
    onSubmitMeta: defaultMeta,
    onSubmit: async ({ value, meta }) => {
      // Hacer algo con los valores pasados mediante handleSubmit
      console.log(`Selected action - ${meta.submitAction}`, value)
    },
  })

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault()
        e.stopPropagation()
      }}
    >
      {/* ... */}
      <button
        type="submit"
        // Sobrescribe el valor predeterminado especificado en onSubmitMeta
        onClick={() => form.handleSubmit({ submitAction: 'continue' })}
      >
        Enviar y continuar
      </button>
      <button
        type="submit"
        onClick={() => form.handleSubmit({ submitAction: 'backToMenu' })}
      >
        Enviar y volver al menú
      </button>
    </form>
  )
}
```

## Transformar datos con Esquemas Estándar (Standard Schemas)

Aunque Tanstack Form proporciona [soporte para Esquemas Estándar (Standard Schemas)](./validation.md) para validación, no conserva los datos de salida del Esquema.

El valor pasado a la función `onSubmit` siempre serán los datos de entrada. Para recibir los datos de salida de un Esquema Estándar, analícelos en la función `onSubmit`:

```tsx
const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form utiliza el tipo de entrada de los Esquemas Estándar
const defaultValues: z.input<typeof schema> = {
  age: '13',
}

const form = useForm({
  defaultValues,
  validators: {
    onChange: schema,
  },
  onSubmit: ({ value }) => {
    const inputAge: string = value.age
    // Pásalo por el esquema para obtener el valor transformado
    const result = schema.parse(value)
    const outputAge: number = result.age
  },
})
```
