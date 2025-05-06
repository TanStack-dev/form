---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-05-06T20:21:34.014Z'
id: listeners
title: Side effects for event triggers
---

Für Situationen, in denen Sie auf Trigger "einwirken" oder "reagieren" möchten, gibt es die Listener-API (Listener API). Beispielsweise, wenn Sie als Entwickler ein Formularfeld zurücksetzen möchten, weil sich ein anderes Feld geändert hat, würden Sie die Listener-API verwenden.

Stellen Sie sich folgenden Benutzerablauf vor:

- Der Benutzer wählt ein Land aus einer Dropdown-Liste aus.
- Der Benutzer wählt dann eine Provinz aus einer anderen Dropdown-Liste aus.
- Der Benutzer ändert das ausgewählte Land zu einem anderen.

In diesem Beispiel muss die ausgewählte Provinz zurückgesetzt werden, wenn der Benutzer das Land ändert, da sie nicht mehr gültig ist. Mit der Listener-API können wir uns auf das `onChange`-Event abonnieren und ein Reset für das Feld "province" auslösen, wenn der Listener aktiviert wird.

Events, auf die "gelauscht" werden kann, sind:

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
