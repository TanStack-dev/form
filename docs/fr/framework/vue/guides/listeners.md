---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-05-06T20:25:44.842Z'
id: listeners
title: Side effects for event triggers
---

Pour les situations où vous souhaitez "affecter" ou "réagir" à des déclencheurs, il existe l'API d'écoute (listener API). Par exemple, si vous, en tant que développeur, souhaitez réinitialiser un champ de formulaire suite à la modification d'un autre champ, vous utiliserez l'API d'écoute.

Imaginez le flux utilisateur suivant :

- L'utilisateur sélectionne un pays dans une liste déroulante.
- L'utilisateur sélectionne ensuite une province dans une autre liste déroulante.
- L'utilisateur modifie le pays sélectionné pour un autre.

Dans cet exemple, lorsque l'utilisateur change le pays, la province sélectionnée doit être réinitialisée car elle n'est plus valide. Avec l'API d'écoute, nous pouvons nous abonner à l'événement `onChange` et déclencher une réinitialisation du champ "province" lorsque l'écouteur est activé.

Les événements pouvant être "écoutés" sont :

- `onChange`
- `onBlur`
- `onMount`
- `onSubmit`

```vue
<script setup>
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    country: '',
    province: '',
  },
  // ...
})
</script>

<template>
  <div>
    <form.Field
      name="country"
      :listeners="{
        onChange: ({ value }) => {
          console.log(`Country changed to: ${value}, resetting province`)
          form.setFieldValue('province', '')
        },
      }"
    >
      <template v-slot="{ field }">
        <input
          :value="field.state.value"
          @input="(e) => field.handleChange(e.target.value)"
        />
      </template>
    </form.Field>

    <form.Field name="province">
      <template v-slot="{ field }">
        <input
          :value="field.state.value"
          @input="(e) => field.handleChange(e.target.value)"
        />
      </template>
    </form.Field>
  </div>
</template>
```
