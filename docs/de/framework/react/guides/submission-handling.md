---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-04-30T21:53:39.530Z'
id: submission-handling
title: Behandeln von Formularübermittlungen
---

## Zusätzliche Daten an die Übermittlungsverarbeitung übergeben

Es kann verschiedene Arten von Übermittlungsverhalten geben, zum Beispiel das Zurückkehren zu einer anderen Seite oder das Verbleiben auf dem Formular. Dies kann durch die Angabe der Eigenschaft `onSubmitMeta` erreicht werden. Diese Metadaten werden an die `onSubmit`-Funktion übergeben.

> Hinweis: Wenn `form.handleSubmit()` ohne Metadaten aufgerufen wird, werden die angegebenen Standardwerte verwendet.

```tsx
import { useForm } from '@tanstack/react-form'

type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null
}

// Metadaten sind nicht erforderlich, um form.handleSubmit() aufzurufen.
// Legen Sie fest, welche Werte als Standard verwendet werden sollen, wenn keine Metadaten übergeben werden
const defaultMeta: FormMeta = {
  submitAction: null,
}

function App() {
  const form = useForm({
    defaultValues: {
      data: '',
    },
    // Definieren Sie, welche Metawerte bei der Übermittlung erwartet werden
    onSubmitMeta: defaultMeta,
    onSubmit: async ({ value, meta }) => {
      // Machen Sie etwas mit den über handleSubmit übergebenen Werten
      console.log(`Ausgewählte Aktion - ${meta.submitAction}`, value)
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

## Daten mit Standard-Schemas transformieren

Während Tanstack Form [Standard-Schema-Unterstützung](./validation.md) für die Validierung bietet, werden die Ausgabedaten des Schemas nicht beibehalten.

Der an die `onSubmit`-Funktion übergebene Wert entspricht immer den Eingabedaten. Um die Ausgabedaten eines Standard-Schemas zu erhalten, müssen Sie diese in der `onSubmit`-Funktion parsen:

```tsx
const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form verwendet den Eingabetyp von Standard-Schemas
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
    // Führen Sie die Daten durch das Schema, um den transformierten Wert zu erhalten
    const result = schema.parse(value)
    const outputAge: number = result.age
  },
})
```
