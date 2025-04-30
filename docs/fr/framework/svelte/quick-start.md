---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:34:50.325Z'
id: quick-start
title: Démarrage rapide
---

# Démarrage rapide

Le strict minimum pour commencer avec TanStack Form est de créer un formulaire et d'y ajouter un champ. Notez que cet exemple n'inclut aucune validation ni gestion d'erreurs... pour l'instant.

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      fullName: '',
    },
    onSubmit: async ({ value }) => {
      // Faire quelque chose avec les données du formulaire
      console.log(value)
    },
  }))
</script>

<div>
  <h1>Exemple de formulaire simple</h1>
  <form
    onsubmit={(e) => {
      e.preventDefault()
      e.stopPropagation()
      form.handleSubmit()
    }}
  >
    <div>
      <form.Field name="fullName">
        {#snippet children(field)}
          <input
            name={field.name}
            value={field.state.value}
            onblur={field.handleBlur}
            oninput={(e) => field.handleChange(e.target.value)}
          />
        {/snippet}
      </form.Field>
    </div>
    <button type="submit">Soumettre</button>
  </form>
</div>
```

À partir de là, vous serez prêt à explorer toutes les autres fonctionnalités de TanStack Form !
