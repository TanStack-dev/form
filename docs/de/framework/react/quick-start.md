---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T21:53:12.320Z'
id: quick-start
title: Schnellstart
---

# Quick Start

TanStack Form unterscheidet sich von den meisten Formularbibliotheken, die Sie bisher verwendet haben. Es wurde für den großflächigen Produktionseinsatz entwickelt, mit Fokus auf Typsicherheit, Leistung und Komposition für ein unvergleichliches Entwicklererlebnis.

Daher haben wir [eine Philosophie zur Nutzung der Bibliothek](/form/latest/docs/philosophy) entwickelt, die Skalierbarkeit und langfristiges Entwicklererlebnis über kurze und teilbare Code-Snippets stellt.

Hier ist ein Beispiel für ein Formular, das viele unserer Best Practices befolgt und Ihnen ermöglicht, selbst hochkomplexe Formulare schnell zu entwickeln, nachdem Sie sich kurz eingearbeitet haben:

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { createFormHook, createFormHookContexts } from '@tanstack/react-form'
// Formularkomponenten, die Ereignisse aus dem Form-Hook vorab binden; siehe unseren "Form Composition"-Guide für mehr
import { TextField, NumberField, SubmitButton } from '~our-app/ui-library'
// Wir unterstützen auch Valibot, ArkType und jede andere Standard-Schema-Bibliothek
import { z } from 'zod'

const { fieldContext, formContext } = createFormHookContexts()

// Ermöglicht uns, Komponenten an das Formular zu binden, um Typsicherheit zu erhalten, aber Boilerplate im Produktivbetrieb zu reduzieren
// Einmal definieren, um einen Generator konsistenter Formularinstanzen in Ihrer App zu haben
const { useAppForm } = createFormHook({
  fieldComponents: {
    TextField,
    NumberField,
  },
  formComponents: {
    SubmitButton,
  },
  fieldContext,
  formContext,
})

const PeoplePage = () => {
  const form = useAppForm({
    defaultValues: {
      username: '',
      age: 0,
    },
    validators: {
      // Schema oder Funktion zur Validierung übergeben
      onChange: z.object({
        username: z.string(),
        age: z.number().min(13),
      }),
    },
    onSubmit: ({ value }) => {
      // Mit den Formulardaten etwas tun
      alert(JSON.stringify(value, null, 2))
    },
  })

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault()
        form.handleSubmit()
      }}
    >
      <h1>Persönliche Informationen</h1>
      {/* Komponenten sind an `form` und `field` gebunden, um maximale Typsicherheit zu gewährleisten */}
      {/* `form.AppField` verwenden, um eine an ein einzelnes Feld gebundene Komponente zu rendern */}
      <form.AppField
        name="username"
        children={(field) => <field.TextField label="Vollständiger Name" />}
      />
      {/* Die "name"-Eigenschaft wirft einen TypeScript-Fehler, wenn sie falsch geschrieben ist */}
      <form.AppField
        name="age"
        children={(field) => <field.NumberField label="Alter" />}
      />
      {/* Komponenten in `form.AppForm` haben Zugriff auf den Formularkontext */}
      <form.AppForm>
        <form.SubmitButton />
      </form.AppForm>
    </form>
  )
}

const rootElement = document.getElementById('root')!
ReactDOM.createRoot(rootElement).render(<PeoplePage />)
```

Während wir generell empfehlen, `createFormHook` für langfristig reduzierten Boilerplate zu verwenden, unterstützen wir auch Einzelkomponenten und andere Verhaltensweisen mit `useForm` und `form.Field`:

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { useForm } from '@tanstack/react-form'

const PeoplePage = () => {
  const form = useForm({
    defaultValues: {
      username: '',
      age: 0,
    },
    onSubmit: ({ value }) => {
      // Mit den Formulardaten etwas tun
      alert(JSON.stringify(value, null, 2))
    },
  })

  return (
    <form.Field
      name="age"
      validators={{
        // Wir können zwischen formularweiten und feld-spezifischen Validatoren wählen
        onChange: ({ value }) =>
          value > 13 ? undefined : 'Muss 13 oder älter sein',
      }}
      children={(field) => (
        <>
          <input
            name={field.name}
            value={field.state.value}
            onBlur={field.handleBlur}
            type="number"
            onChange={(e) => field.handleChange(e.target.valueAsNumber)}
          />
          {!field.state.meta.isValid && (
            <em>{field.state.meta.errors.join(',')}</em>
          )}
        </>
      )}
    />
  )
}

const rootElement = document.getElementById('root')!
ReactDOM.createRoot(rootElement).render(<PeoplePage />)
```

Alle Eigenschaften von `useForm` können in `useAppForm` verwendet werden, und alle Eigenschaften von `form.Field` können in `form.AppField` genutzt werden.
