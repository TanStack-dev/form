---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:36:22.201Z'
id: linked-fields
title: Champs liés
---

# Lier deux champs de formulaire ensemble

Vous pourriez avoir besoin de lier deux champs ensemble ; lorsqu'un champ est validé en fonction de la modification de la valeur d'un autre champ.  
Un cas d'usage courant est lorsque vous avez à la fois un champ `password` et un champ `confirm_password`,  
où vous souhaitez que `confirm_password` affiche une erreur lorsque sa valeur ne correspond pas à celle de `password`,  
peu importe quel champ a déclenché le changement de valeur.

Imaginez le flux utilisateur suivant :

- L'utilisateur modifie le champ de confirmation de mot de passe.
- L'utilisateur modifie le champ de mot de passe principal.

Dans cet exemple, le formulaire affichera toujours des erreurs,  
car la validation du champ "confirm password" n'a pas été relancée pour marquer la valeur comme acceptée.

Pour résoudre ce problème, nous devons nous assurer que la validation du "confirm password" est relancée lorsque le champ password est mis à jour.  
Pour ce faire, vous pouvez ajouter une propriété `onChangeListenTo` au champ `confirm_password`.

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    password: '',
    confirm_password: '',
  },
  // ...
})
</script>

<template>
  <div>
    <form @submit.prevent.stop="form.handleSubmit">
      <div>
        <form.Field name="password">
          <template v-slot="{ field }">
            <div>Password:</div>
            <input
              :value="field.state.value"
              @input="
                (e) => field.handleChange((e.target as HTMLInputElement).value)
              "
            />
          </template>
        </form.Field>
        <form.Field
          name="confirm_password"
          :validators="{
            onChangeListenTo: ['password'],
            onChange: ({ value, fieldApi }) => {
              if (value !== fieldApi.form.getFieldValue('password')) {
                return 'Passwords do not match'
              }
              return undefined
            },
          }"
        >
          <template v-slot="{ field }">
            <div>Confirm Password:</div>
            <input
              :value="field.state.value"
              @input="
                (e) => field.handleChange((e.target as HTMLInputElement).value)
              "
            />
            <div v-for="(err, index) in field.state.meta.errors" :key="index">
              {{ err }}
            </div>
          </template>
        </form.Field>
      </div>
      <button type="submit">Submit</button>
    </form>
  </div>
</template>
```

Ceci fonctionne également avec la propriété `onBlurListenTo`, qui relancera la validation lorsque le champ perd le focus.
