---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:22:37.353Z'
id: submission-handling
title: Submission handling
---

## Zusätzliche Daten an die Übermittlungsverarbeitung übergeben

Es kann verschiedene Arten von Übermittlungsverhalten geben, zum Beispiel das Zurückkehren zu einer anderen Seite oder das Verbleiben auf dem Formular.  
Dies können Sie erreichen, indem Sie die Eigenschaft `onSubmitMeta` angeben. Diese Metadaten werden an die `onSubmit`-Funktion übergeben.

> Hinweis: Wenn `form.handleSubmit()` ohne Metadaten aufgerufen wird, werden die angegebenen Standardwerte verwendet.

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null
}

// Metadaten sind nicht erforderlich, um form.handleSubmit() aufzurufen.
// Legen Sie fest, welche Werte als Standard verwendet werden sollen, wenn keine Metadaten übergeben werden
const defaultMeta: FormMeta = {
  submitAction: null,
}

const form = useForm({
  defaultValues: {
    data: '',
  },
  // Definieren Sie, welche Metadaten bei der Übermittlung erwartet werden
  onSubmitMeta: defaultMeta,
  onSubmit: async ({ value, meta }) => {
    // Verarbeiten Sie die über handleSubmit übergebenen Werte
    console.log(`Ausgewählte Aktion - ${meta.submitAction}`, value)
  },
})
</script>

<template>
  <form @submit.prevent.stop="">
    <button
      type="submit"
      @click="
        () => {
          // Überschreibt den in onSubmitMeta angegebenen Standardwert
          form.handleSubmit({ submitAction: 'continue' })
        }
      "
    >
      Absenden und fortfahren
    </button>
    <button
      type="submit"
      @click="() => form.handleSubmit({ submitAction: 'backToMenu' })"
    >
      Absenden und zurück zum Menü
    </button>
  </form>
</template>
```

## Daten mit Standard-Schemas transformieren

Während Tanstack Form [Standard-Schema-Unterstützung](./validation.md) für die Validierung bietet, werden die Ausgabedaten des Schemas nicht beibehalten.

Der an die `onSubmit`-Funktion übergebene Wert entspricht immer den Eingabedaten. Um die Ausgabedaten eines Standard-Schemas zu erhalten, parsen Sie diese in der `onSubmit`-Funktion:

```vue
<script setup lang="ts">
import { z } from 'zod'
import { useForm } from '@tanstack/vue-form'

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
    // Verarbeiten Sie die Daten durch das Schema, um den transformierten Wert zu erhalten
    const result = schema.parse(value)
    const outputAge: number = result.age
  },
})
</script>
<template>
  <!-- ... -->
</template>
```
