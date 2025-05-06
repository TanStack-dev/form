---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:24:29.779Z'
id: submission-handling
title: Submission handling
---

## Transmission de données supplémentaires lors de la gestion de la soumission

Vous pouvez avoir plusieurs types de comportements de soumission, par exemple, revenir à une autre page ou rester sur le formulaire.  
Vous pouvez accomplir cela en spécifiant la propriété `onSubmitMeta`. Ces métadonnées seront transmises à la fonction `onSubmit`.

> Remarque : si `form.handleSubmit()` est appelé sans métadonnées, il utilisera les valeurs par défaut fournies.

```tsx
import { createForm } from '@tanstack/solid-form'

type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null
}

// Les métadonnées ne sont pas obligatoires pour appeler form.handleSubmit().
// Spécifiez les valeurs à utiliser par défaut si aucune métadonnée n'est transmise
const defaultMeta: FormMeta = {
  submitAction: null,
}

export default function App() {
  const form = createForm(() => ({
    defaultValues: {
      data: '',
    },
    // Définissez les valeurs de métadonnées attendues lors de la soumission
    onSubmitMeta: defaultMeta,
    onSubmit: async ({ value, meta }) => {
      // Faites quelque chose avec les valeurs transmises via handleSubmit
      console.log(`Selected action - ${meta.submitAction}`, value)
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
        // Remplace la valeur par défaut spécifiée dans onSubmitMeta
        onClick={() => form.handleSubmit({ submitAction: 'continue' })}
      >
        Soumettre et continuer
      </button>
      <button
        type="submit"
        onClick={() => form.handleSubmit({ submitAction: 'backToMenu' })}
      >
        Soumettre et revenir au menu
      </button>
    </form>
  )
}
```

## Transformation des données avec les schémas standard (Standard Schemas)

Bien que Tanstack Form prenne en charge les [schémas standard (Standard Schemas)](./validation.md) pour la validation, il ne conserve pas les données de sortie du schéma.

La valeur transmise à la fonction `onSubmit` sera toujours les données d'entrée. Pour recevoir les données de sortie d'un schéma standard, analysez-les dans la fonction `onSubmit` :

```tsx
import { createForm } from '@tanstack/solid-form'
import { z } from 'zod'

// ...

const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form utilise le type d'entrée des schémas standard
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
    // Passez les données à travers le schéma pour obtenir la valeur transformée
    const result = schema.parse(value)
    const outputAge: number = result.age
  },
}))
```
