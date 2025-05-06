---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:23:28.745Z'
id: submission-handling
title: Submission handling
---

## Übergeben zusätzlicher Daten an die Submission-Handling (Submission Handling)

Es kann vorkommen, dass Sie verschiedene Arten von Submission-Verhalten (Submission Behaviour) benötigen, beispielsweise das Zurückkehren zu einer anderen Seite oder das Verbleiben auf dem Formular. Dies können Sie erreichen, indem Sie die Eigenschaft `onSubmitMeta` angeben. Diese Metadaten werden an die `onSubmit`-Funktion übergeben.

> Hinweis: Wenn `form.handleSubmit()` ohne Metadaten aufgerufen wird, werden die angegebenen Standardwerte verwendet.

```tsx
import { createForm } from '@tanstack/solid-form'

type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null
}

// Metadaten sind nicht erforderlich, um form.handleSubmit() aufzurufen.
// Legen Sie fest, welche Werte als Standard verwendet werden sollen, wenn keine Metadaten übergeben werden
const defaultMeta: FormMeta = {
  submitAction: null,
}

export default function App() {
  const form = createForm(() => ({
    defaultValues: {
      data: '',
    },
    // Definieren Sie, welche Metadaten bei der Submission erwartet werden
    onSubmitMeta: defaultMeta,
    onSubmit: async ({ value, meta }) => {
      // Führen Sie etwas mit den über handleSubmit übergebenen Werten aus
      console.log(`Ausgewählte Aktion - ${meta.submitAction}`, value)
    },
  }))

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
        // Überschreibt den in onSubmitMeta angegebenen Standardwert
        onClick={() => form.handleSubmit({ submitAction: 'continue' })}
      >
        Absenden und fortfahren
      </button>
      <button
        type="submit"
        onClick={() => form.handleSubmit({ submitAction: 'backToMenu' })}
      >
        Absenden und zurück zum Menü
      </button>
    </form>
  )
}
```

## Transformieren von Daten mit Standard-Schemas (Standard Schemas)

Während Tanstack Form [Unterstützung für Standard-Schemas](./validation.md) zur Validierung bietet, werden die Ausgabedaten des Schemas nicht beibehalten.

Der an die `onSubmit`-Funktion übergebene Wert entspricht immer den Eingabedaten. Um die Ausgabedaten eines Standard-Schemas zu erhalten, müssen Sie diese in der `onSubmit`-Funktion parsen:

```tsx
import { createForm } from '@tanstack/solid-form'
import { z } from 'zod'

// ...

const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form verwendet den Eingabetyp von Standard-Schemas
const defaultValues: z.input<typeof schema> = {
  age: '13',
}

const form = createForm(() => ({
  defaultValues,
  validators: {
    onChange: schema,
  },
  onSubmit: ({ value }) => {
    const inputAge: string = value.age
    // Führen Sie das Parsing durch das Schema durch, um den transformierten Wert zu erhalten
    const result = schema.parse(value)
    const outputAge: number = result.age
  },
}))
```
