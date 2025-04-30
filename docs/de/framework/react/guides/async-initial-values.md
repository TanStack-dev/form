---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:53:12.038Z'
id: async-initial-values
title: Asynchrone Anfangswerte
---

## Asynchrone Initialwerte

Angenommen, Sie möchten einige Daten von einer API abrufen und sie als Initialwerte eines Formulars verwenden.

Obwohl dieses Problem oberflächlich betrachtet einfach klingt, gibt es versteckte Komplexitäten, an die Sie möglicherweise noch nicht gedacht haben.

Beispielsweise möchten Sie vielleicht einen Ladeindikator anzeigen, während die Daten abgerufen werden, oder Fehler elegant behandeln.  
Ebenso könnten Sie nach einer Möglichkeit suchen, die Daten zwischenzuspeichern, damit Sie sie nicht jedes Mal abrufen müssen, wenn das Formular gerendert wird.

Während wir viele dieser Funktionen von Grund auf implementieren könnten, würde das Ergebnis letztendlich einem anderen Projekt ähneln, das wir pflegen: [TanStack Query](https://tanstack.com/query).

Daher zeigt Ihnen diese Anleitung, wie Sie TanStack Form mit TanStack Query kombinieren können, um das gewünschte Verhalten zu erreichen.

## Grundlegende Verwendung

```tsx
import { useForm } from '@tanstack/react-form'
import { useQuery } from '@tanstack/react-query'

export default function App() {
  const {data, isLoading} = useQuery({
    queryKey: ['data'],
    queryFn: async () => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return {firstName: 'FirstName', lastName: "LastName"}
    }
  })

  const form = useForm({
    defaultValues: {
      firstName: data?.firstName ?? '',
      lastName: data?.lastName ?? '',
    },
    onSubmit: async ({ value }) => {
      // Do something with form data
      console.log(value)
    },
  })

  if (isLoading) return <p>Loading..</p>

  return (
    // ...
  )
}
```

Dies zeigt einen Ladeindikator an, bis die Daten abgerufen wurden, und rendert dann das Formular mit den abgerufenen Daten als Initialwerte.
