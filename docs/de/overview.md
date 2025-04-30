---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T21:53:12.553Z'
id: overview
title: Übersicht
---

# Übersicht

TanStack Form ist die ultimative Lösung für die Handhabung von Formularen in Webanwendungen und bietet einen leistungsstarken und flexiblen Ansatz für das Formularmanagement. Mit erstklassiger TypeScript-Unterstützung, Headless-UI-Komponenten (UI-Komponenten ohne vorgefertigtes Styling) und einem framework-agnostischen Design optimiert es die Formularverarbeitung und gewährleistet eine nahtlose Erfahrung über verschiedene Frontend-Frameworks hinweg.

## Motivation

Die meisten Webframeworks bieten keine umfassende Lösung für die Formularverarbeitung, sodass Entwickler eigene Implementierungen erstellen oder auf weniger leistungsfähige Bibliotheken zurückgreifen müssen. Dies führt oft zu Inkonsistenzen, schlechter Performance und erhöhtem Entwicklungsaufwand. TanStack Form zielt darauf ab, diese Herausforderungen zu bewältigen, indem es eine All-in-One-Lösung für das Formularmanagement bereitstellt, die sowohl leistungsstark als auch benutzerfreundlich ist.

Mit TanStack Form können Entwickler häufige Herausforderungen im Zusammenhang mit Formularen bewältigen, wie zum Beispiel:

- Reaktives Data Binding (Datenbindung) und State Management (Zustandsverwaltung)
- Komplexe Validierung und Fehlerbehandlung
- Barrierefreiheit (Accessibility) und responsives Design
- Internationalisierung (Internationalization) und Lokalisierung (Localization)
- Plattformübergreifende Kompatibilität und individuelles Styling

Indem es eine vollständige Lösung für diese Herausforderungen bietet, ermöglicht TanStack Form Entwicklern, robuste und benutzerfreundliche Formulare mit Leichtigkeit zu erstellen.

## Genug geredet, zeig mir endlich etwas Code!

Im folgenden Beispiel sehen Sie TanStack Form in Aktion mit dem React-Framework-Adapter:

[In CodeSandbox öffnen](https://codesandbox.io/s/github/tanstack/form/tree/main/examples/react/simple)

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

## Du hast mich überzeugt, was nun?

- Lernen Sie TanStack Form in Ihrem eigenen Tempo mit unserer umfassenden [Walkthrough-Anleitung](../installation) und [API-Referenz](../reference/classes/formapi)
