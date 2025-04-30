---
source-updated-at: '2025-03-31T17:05:18.000Z'
translation-updated-at: '2025-04-30T21:52:41.570Z'
id: quick-start
title: Schnellstart
---

# Quick Start

Das absolute Minimum, um mit TanStack Form zu beginnen, ist das Erstellen eines Formulars und das Hinzufügen eines Feldes. Beachten Sie, dass dieses Beispiel noch keine Validierung oder Fehlerbehandlung enthält... bisher.

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

Von hier aus können Sie alle anderen Funktionen von TanStack Form erkunden!
