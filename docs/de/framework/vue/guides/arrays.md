---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:54:31.649Z'
id: arrays
title: Arrays
---

# Arrays

TanStack Form unterstützt Arrays als Werte in einem Formular, einschließlich Unterobjekt-Werten innerhalb eines Arrays.

## Grundlegende Verwendung

Um ein Array zu verwenden, können Sie `field.state.value` auf einen Array-Wert in Verbindung mit [`Index` von `solid-js`](https://www.solidjs.com/tutorial/flow_index) nutzen:

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    people: [] as Array<{ age: number; name: string }>,
  },
  onSubmit: ({ value }) => alert(JSON.stringify(value)),
})
</script>

<template>
  <form.Field name="people">
    <template v-slot="{ field, state }">
      <div>
        <form.Field
          v-for="(_, i) of field.state.value"
          :key="i"
          :name="`people[${i}].name`"
        >
          <template v-slot="{ field: subField, state }">
            <!-- ... -->
          </template>
        </form.Field>
      </div>
    </template>
  </form.Field>
</template>
```

Dies generiert den gemappten Slot jedes Mal, wenn Sie `pushValue` auf `field` ausführen:

```vue
<button @click="field.pushValue({ name: '', age: 0 })" type="button">
  Person hinzufügen
</button>
```

Schließlich können Sie ein Unterfeld wie folgt verwenden:

```vue
<form.Field
  v-for="(_, i) of field.state.value"
  :key="i"
  :name="`people[${i}].name`"
>
  <template v-slot="{ field: subField, state }">
    <div>
      <label>
        <div>Name für Person {{ i }}</div>
        <input
          :value="subField.state.value"
          @input="
          (e) =>
          subField.handleChange(
            (e.target as HTMLInputElement).value,
          )
          "
        />
      </label>
    </div>
  </template>
</form.Field>
```

## Vollständiges Beispiel

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    people: [] as Array<{ age: number; name: string }>,
  },
  onSubmit: ({ value }) => alert(JSON.stringify(value)),
})
</script>

<template>
  <form
    @submit="
      (e) => {
        e.preventDefault()
        e.stopPropagation()
        form.handleSubmit()
      }
    "
  >
    <div>
      <form.Field name="people">
        <template v-slot="{ field, state }">
          <div>
            <form.Field
              v-for="(_, i) of field.state.value"
              :key="i"
              :name="`people[${i}].name`"
            >
              <template v-slot="{ field: subField, state }">
                <div>
                  <label>
                    <div>Name für Person {{ i }}</div>
                    <input
                      :value="subField.state.value"
                      @input="
                        (e) =>
                          subField.handleChange(
                            (e.target as HTMLInputElement).value,
                          )
                      "
                    />
                  </label>
                </div>
              </template>
            </form.Field>

            <button
              @click="field.pushValue({ name: '', age: 0 })"
              type="button"
            >
              Person hinzufügen
            </button>
          </div>
        </template>
      </form.Field>
    </div>
    <form.Subscribe>
      <template v-slot="{ canSubmit, isSubmitting }">
        <button type="submit" :disabled="!canSubmit">
          {{ isSubmitting ? '...' : 'Absenden' }}
        </button>
      </template>
    </form.Subscribe>
  </form>
</template>
```
