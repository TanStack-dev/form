---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:56:05.952Z'
id: async-initial-values
title: Asynchrone Anfangswerte
---

## Motivation

Die meisten Web-Frameworks bieten keine umfassende Lösung für die Formularverarbeitung, was Entwickler dazu zwingt, eigene Implementierungen zu erstellen oder auf weniger leistungsfähige Bibliotheken zurückzugreifen. Dies führt oft zu Inkonsistenzen, schlechter Performance und erhöhtem Entwicklungsaufwand. TanStack Form zielt darauf ab, diese Herausforderungen zu lösen, indem es eine All-in-One-Lösung für die Formularverwaltung bietet, die sowohl leistungsstark als auch benutzerfreundlich ist.

Mit TanStack Form können Entwickler häufige Herausforderungen im Zusammenhang mit Formularen bewältigen, wie zum Beispiel:

- Reaktives Data Binding und State Management
- Komplexe Validierung und Fehlerbehandlung
- Barrierefreiheit und responsives Design
- Internationalisierung und Lokalisierung
- Plattformübergreifende Kompatibilität und individuelles Styling

Indem es eine vollständige Lösung für diese Herausforderungen bietet, ermöglicht TanStack Form Entwicklern, robuste und benutzerfreundliche Formulare mit Leichtigkeit zu erstellen.

## Grundlegende Verwendung

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'
  import { createQuery } from '@tanstack/svelte-query'

    const { data, isLoading } = createQuery(() => ({
      queryKey: ['data'],
      queryFn: async () => {
        await new Promise((resolve) => setTimeout(resolve, 1000))
        return { firstName: 'FirstName', lastName: 'LastName' }
      },
    }))

    const form = createForm(() => ({
      defaultValues: {
        firstName: $data?.firstName ?? '',
        lastName: $data?.lastName ?? '',
      },
      onSubmit: async ({ value }) => {
        // Hier kann mit den Formulardaten gearbeitet werden
        console.log(value)
      },
    }))
</script>

{#if $isLoading}
  <p>Ladevorgang...</p>
{:else}
  <!-- Formular... -->
{/if}
```

Dies zeigt einen Ladeindikator an, bis die Daten abgerufen wurden, und rendert anschließend das Formular mit den abgerufenen Daten als Initialwerte.
