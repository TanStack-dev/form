---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:34:37.663Z'
id: quick-start
title: Démarrage rapide
---

# Démarrage rapide

Le strict minimum pour commencer avec TanStack Form est de créer un formulaire et d'y ajouter un champ. Gardez à l'esprit que cet exemple n'inclut aucune validation ni gestion d'erreurs... pour l'instant.

```tsx
import { createForm } from '@tanstack/solid-form'

function App() {
  const form = createForm(() => ({
    defaultValues: {
      fullName: '',
    },
    onSubmit: async ({ value }) => {
      // Faire quelque chose avec les données du formulaire
      console.log(value)
    },
  }))

  return (
    <div>
      <h1>Exemple de formulaire simple</h1>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          form.handleSubmit()
        }}
      >
        <div>
          <form.Field
            name="fullName"
            children={(field) => (
              <input
                name={field().name}
                value={field().state.value}
                onBlur={field().handleBlur}
                onInput={(e) => field().handleChange(e.target.value)}
              />
            )}
          />
        </div>
        <button type="submit">Envoyer</button>
      </form>
    </div>
  )
}
```

À partir de là, vous serez prêt à explorer toutes les autres fonctionnalités de TanStack Form !
