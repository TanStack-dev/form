---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:55:23.776Z'
id: basic-concepts
title: Grundkonzepte
---

Diese Seite führt die grundlegenden Konzepte und Begriffe ein, die in der `@tanstack/vue-form`-Bibliothek verwendet werden. Wenn Sie sich mit diesen Konzepten vertraut machen, können Sie die Bibliothek besser verstehen und damit arbeiten.

## Formularoptionen

Sie können Optionen für Ihr Formular erstellen, damit sie zwischen mehreren Formularen gemeinsam genutzt werden können, indem Sie die Funktion `formOptions` verwenden.

Beispiel:

```ts
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## Formularinstanz

Eine Formularinstanz ist ein Objekt, das ein einzelnes Formular repräsentiert und Methoden und Eigenschaften für die Arbeit mit dem Formular bereitstellt. Sie erstellen eine Formularinstanz mit der Funktion `useForm`. Die Funktion akzeptiert ein Objekt mit einer `onSubmit`-Funktion, die aufgerufen wird, wenn das Formular abgesendet wird.

```js
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // Mit den Formulardaten etwas tun
    console.log(value)
  },
})
```

Sie können auch eine Formularinstanz erstellen, ohne `formOptions` zu verwenden, indem Sie die eigenständige `useForm`-API nutzen:

```ts
const form = useForm({
  onSubmit: async ({ value }) => {
    // Mit den Formulardaten etwas tun
    console.log(value)
  },
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## Feld

Ein Feld repräsentiert ein einzelnes Formulareingabeelement, wie z.B. ein Texteingabefeld oder eine Checkbox. Felder werden mit der `form.Field`-Komponente erstellt, die von der Formularinstanz bereitgestellt wird. Die Komponente akzeptiert eine `name`-Prop, die mit einem Schlüssel in den Standardwerten des Formulars übereinstimmen sollte. Sie akzeptiert auch einen Scoped Slot, der durch die `v-slot`-Direktive definiert wird und ein `field`-Objekt als Argument erhält.

Beispiel:

```vue
<template>
  <!-- ... -->
  <form.Field name="fullName">
    <template v-slot="{ field }">
      <input
        :name="field.name"
        :value="field.state.value"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange(e.target.value)"
      />
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## Feldzustand

Jedes Feld hat seinen eigenen Zustand, der seinen aktuellen Wert, den Validierungsstatus, Fehlermeldungen und andere Metadaten enthält. Sie können den Zustand eines Felds über die Eigenschaft `field.state` abrufen.

Beispiel:

```js
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Es gibt drei Feldzustände, die sehr nützlich sein können, um zu sehen, wie der Benutzer mit einem Feld interagiert. Ein Feld ist _"berührt" (touched)_, wenn der Benutzer hineinklickt/-tabuliert, _"unverändert" (pristine)_, bis der Benutzer den Wert ändert, und _"verändert" (dirty)_, nachdem der Wert geändert wurde. Sie können diese Zustände über die Flags `isTouched`, `isPristine` und `isDirty` überprüfen, wie unten gezeigt.

```js
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Feldzustände](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## Feld-API

Die Feld-API ist ein Objekt, das von einem Scoped Slot über die `v-slot`-Direktive bereitgestellt wird. Dieser Slot erhält ein Argument namens `field`, das Methoden und Eigenschaften für die Arbeit mit dem Feldzustand bereitstellt.

Beispiel:

```vue
<template v-slot="{ field }">
  <input
    :name="field.name"
    :value="field.state.value"
    @blur="field.handleBlur"
    @input="(e) => field.handleChange(e.target.value)"
  />
</template>
```

## Validierung

`@tanstack/vue-form` bietet sowohl synchrone als auch asynchrone Validierung von Haus aus. Validierungsfunktionen können der `form.Field`-Komponente über die `validators`-Prop übergeben werden.

Beispiel:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="firstName"
    :validators="{
      onChange: ({ value }) =>
        !value
          ? `Ein Vorname ist erforderlich`
          : value.length < 3
            ? `Der Vorname muss mindestens 3 Zeichen lang sein`
            : undefined,
      onChangeAsync: async ({ value }) => {
        await new Promise((resolve) => setTimeout(resolve, 1000))
        return (
          value.includes('error') && '„error“ ist im Vornamen nicht erlaubt'
        )
      },
    }"
  >
    <template v-slot="{ field }">
      <input
        :value="field.state.value"
        @input="(e) => field.handleChange(e.target.value)"
        @blur="field.handleBlur"
      />
      <FieldInfo :field="field" />
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## Validierung mit Standard-Schema-Bibliotheken

Zusätzlich zu manuell erstellten Validierungsoptionen unterstützen wir auch die [Standard-Schema](https://github.com/standard-schema/standard-schema)-Spezifikation.

Sie können ein Schema mit einer der Bibliotheken definieren, die die Spezifikation implementieren, und es einem Formular- oder Feldvalidierer übergeben.

Unterstützte Bibliotheken sind:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'
import { z } from 'zod'

const form = useForm({
  // ...
})

const onChangeFirstName = z.string().refine(
  async (value) => {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return !value.includes('error')
  },
  {
    message: '„error“ ist im Vornamen nicht erlaubt',
  },
)
</script>

<template>
  <!-- ... -->
  <form.Field
    name="firstName"
    :validators="{
      onChange: z
        .string()
        .min(3, 'Der Vorname muss mindestens 3 Zeichen lang sein'),
      onChangeAsyncDebounceMs: 500,
      onChangeAsync: onChangeFirstName,
    }"
  >
    <template v-slot="{ field, state }">
      <label :htmlFor="field.name">Vorname:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).value)"
        @blur="field.handleBlur"
      />
      <FieldInfo :state="state" />
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## Reaktivität

`@tanstack/vue-form` bietet verschiedene Möglichkeiten, um Änderungen am Formular- und Feldzustand zu abonnieren, insbesondere die Methode `form.useStore` und die Komponente `form.Subscribe`. Diese Methoden ermöglichen es Ihnen, die Rendering-Performance Ihres Formulars zu optimieren, indem Komponenten nur bei Bedarf aktualisiert werden.

Beispiel:

```vue
<script setup lang="ts">
// ...
const firstName = form.useStore((state) => state.values.firstName)
</script>

<template>
  <!-- ... -->
  <form.Subscribe>
    <template v-slot="{ canSubmit, isSubmitting }">
      <button type="submit" :disabled="!canSubmit">
        {{ isSubmitting ? '...' : 'Absenden' }}
      </button>
    </template>
  </form.Subscribe>
  <!-- ... -->
</template>
```

## Listener

`@tanstack/vue-form` ermöglicht es Ihnen, auf bestimmte Trigger zu reagieren und sie zu "überwachen", um Nebenwirkungen auszulösen.

Beispiel:

```vue
<template>
  <form.Field
    name="country"
    :listeners="{
      onChange: ({ value }) => {
        console.log(`Land geändert zu: ${value}, Bundesland wird zurückgesetzt`)
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
</template>
```

Weitere Informationen finden Sie unter [Listener](./listeners.md).

Hinweis: Die Verwendung der Methode `form.useField` zur Erzielung von Reaktivität wird nicht empfohlen, da sie gezielt innerhalb der `form.Field`-Komponente verwendet werden sollte. Stattdessen können Sie `form.useStore` verwenden.

## Array-Felder

Array-Felder ermöglichen es Ihnen, eine Liste von Werten innerhalb eines Formulars zu verwalten, wie z.B. eine Liste von Hobbys. Sie können ein Array-Feld mit der `form.Field`-Komponente und der `mode="array"`-Prop erstellen.

Bei der Arbeit mit Array-Feldern können Sie die Methoden `pushValue`, `removeValue`, `swapValues` und `moveValue` verwenden, um Werte im Array hinzuzufügen, zu entfernen und zu vertauschen.

Beispiel:

```vue
<template>
  <form @submit.prevent.stop="form.handleSubmit">
    <form.Field name="hobbies" mode="array">
      <template v-slot="{ field: hobbiesField }">
        <div>
          Hobbys
          <div>
            <div
              v-if="
                Array.isArray(hobbiesField.state.value) &&
                !hobbiesField.state.value.length
              "
            >
              Keine Hobbys gefunden.
            </div>
            <div v-else>
              <div v-for="(_, i) in hobbiesField.state.value" :key="i">
                <form.Field :name="`hobbies[${i}].name`">
                  <template v-slot="{ field }">
                    <div>
                      <label :for="field.name">Name:</label>
                      <input
                        :id="field.name"
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="(e) => field.handleChange(e.target.value)"
                      />
                      <button
                        type="button"
                        @click="hobbiesField.removeValue(i)"
                      >
                        X
                      </button>
                      <FieldInfo :field="field" />
                    </div>
                  </template>
                </form.Field>
                <form.Field :name="`hobbies[${i}].description`">
                  <template v-slot="{ field }">
                    <div>
                      <label :for="field.name">Beschreibung:</label>
                      <input
                        :id="field.name"
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="(e) => field.handleChange(e.target.value)"
                      />
                      <FieldInfo :field="field" />
                    </div>
                  </template>
                </form.Field>
              </div>
            </div>
            <button
              type="button"
              @click="
                hobbiesField.pushValue({
                  name: '',
                  description: '',
                  yearsOfExperience: 0,
                })
              "
            >
              Hobby hinzufügen
            </button>
          </div>
        </div>
      </template>
    </form.Field>
  </form>
</template>
```

Dies sind die grundlegenden Konzepte und Begriffe, die in der `@tanstack/vue-form`-Bibliothek verwendet werden. Wenn Sie diese Konzepte verstehen, können Sie effektiver mit der Bibliothek arbeiten und komplexe Formulare mühelos erstellen.
