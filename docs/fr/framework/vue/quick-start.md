---
source-updated-at: '2025-03-31T17:05:18.000Z'
translation-updated-at: '2025-04-30T21:34:39.287Z'
id: quick-start
title: Démarrage rapide
---

# Démarrage rapide

Le strict minimum pour commencer avec TanStack Form est de créer un formulaire et d'ajouter un champ. Gardez à l'esprit que cet exemple n'inclut aucune validation ou gestion d'erreur... pour l'instant.

```vue
<!-- App.vue -->
<script setup>
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    fullName: '',
  },
  onSubmit: async ({ value }) => {
    // Do something with form data
    console.log(value)
  },
})
</script>

<template>
  <div>
    <form @submit.prevent.stop="form.handleSubmit">
      <div>
        <form.Field name="fullName">
          <template v-slot="{ field }">
            <input
              :name="field.name"
              :value="field.state.value"
              @blur="field.handleBlur"
              @input="(e) => field.handleChange((e.target as HTMLInputElement).value)"
            />
          </template>
        </form.Field>
      </div>
      <button type="submit">Submit</button>
    </form>
  </div>
</template>
```

À partir de là, vous serez prêt·e à explorer toutes les autres fonctionnalités de TanStack Form !
