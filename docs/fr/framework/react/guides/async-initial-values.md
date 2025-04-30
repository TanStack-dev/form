---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:34:58.700Z'
id: async-initial-values
title: Valeurs initiales asynchrones
---

Supposons que vous souhaitiez récupérer des données depuis une API et les utiliser comme valeurs initiales d'un formulaire.

Bien que ce problème semble simple en surface, il cache des complexités auxquelles vous n'avez peut-être pas encore pensé.

Par exemple, vous pourriez vouloir afficher un indicateur de chargement pendant la récupération des données, ou gérer les erreurs de manière élégante.
De même, vous pourriez aussi chercher un moyen de mettre en cache les données pour éviter de les récupérer à chaque rendu du formulaire.

Bien que nous puissions implémenter beaucoup de ces fonctionnalités à partir de zéro, cela finirait par ressembler beaucoup à un autre projet que nous maintenons : [TanStack Query](https://tanstack.com/query).

Ainsi, ce guide vous montre comment combiner TanStack Form avec TanStack Query pour obtenir le comportement souhaité.

## Utilisation de base

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

Cela affichera un indicateur de chargement jusqu'à ce que les données soient récupérées, puis rendra le formulaire avec les données obtenues comme valeurs initiales.
