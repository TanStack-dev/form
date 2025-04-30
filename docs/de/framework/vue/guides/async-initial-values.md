---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:54:08.678Z'
id: async-initial-values
title: Asynchrone Anfangswerte
---

## Asynchrone Initialwerte

Angenommen, Sie möchten Daten von einer API abrufen und als Initialwerte für ein Formular verwenden.

Obwohl dieses Problem oberflächlich betrachtet einfach erscheint, gibt es versteckte Komplexitäten, die Sie möglicherweise noch nicht bedacht haben.

Beispielsweise möchten Sie vielleicht einen Ladeindikator anzeigen, während die Daten abgerufen werden, oder Fehler auf elegante Weise behandeln.

Ebenso könnten Sie nach einer Möglichkeit suchen, die Daten zwischenzuspeichern, um sie nicht bei jedem Rendern des Formulars neu abrufen zu müssen.

Während wir viele dieser Funktionen von Grund auf implementieren könnten, würde das Ergebnis stark einem anderen Projekt ähneln, das wir pflegen: [TanStack Query](https://tanstack.com/query).

Daher zeigt Ihnen diese Anleitung, wie Sie TanStack Form mit TanStack Query kombinieren können, um das gewünschte Verhalten zu erreichen.

## Grundlegende Verwendung

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'
import { useQuery } from '@tanstack/vue-query'
import { watchEffect, reactive } from 'vue'

const { data, isLoading } = useQuery({
  queryKey: ['data'],
  queryFn: async () => {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return { firstName: 'FirstName', lastName: 'LastName' }
  },
})

const firstName = computed(() => data.value?.firstName || '')
const lastName = computed(() => data.value?.lastName || '')

const defaultValues = reactive({
  firstName,
  lastName,
})

const form = useForm({
  defaultValues,
  onSubmit: async ({ value }) => {
    // Hier können Sie mit den Formulardaten arbeiten
    console.log(value)
  },
})
</script>

<template>
  <p v-if="isLoading">Laden..</p>
  <form v-else @submit.prevent.stop="form.handleSubmit">
    <!-- ... -->
  </form>
</template>
```

Dies zeigt einen Ladehinweis an, bis die Daten abgerufen wurden, und rendert anschließend das Formular mit den abgerufenen Daten als Initialwerte.
