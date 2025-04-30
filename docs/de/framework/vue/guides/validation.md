---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T21:56:08.126Z'
id: form-validation
title: Formularvalidierung
---

Im Kern der Funktionalitäten von TanStack Form steht das Konzept der Validierung. TanStack Form macht die Validierung hochgradig anpassbar:

- Sie können steuern, wann die Validierung durchgeführt wird (bei Änderung, bei Eingabe, beim Verlassen des Felds, beim Absenden usw.)
- Validierungsregeln können auf Feldebene oder auf Formularebene definiert werden
- Die Validierung kann synchron oder asynchron erfolgen (z.B. als Ergebnis eines API-Aufrufs)

## Wann wird die Validierung durchgeführt?

Das liegt bei Ihnen! Die `<Field />`-Komponente akzeptiert einige Callbacks als Props, wie z.B. `onChange` oder `onBlur`. Diese Callbacks erhalten den aktuellen Wert des Felds sowie das `fieldAPI`-Objekt, sodass Sie die Validierung durchführen können. Wenn Sie einen Validierungsfehler finden, geben Sie einfach die Fehlermeldung als String zurück, und sie wird in `field.state.meta.errors` verfügbar sein.

Hier ein Beispiel:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

Im obigen Beispiel wird die Validierung bei jedem Tastendruck durchgeführt (`onChange`). Wenn wir stattdessen die Validierung beim Verlassen des Felds durchführen wollten, würden wir den Code wie folgt ändern:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onBlur: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <!-- Wir müssen immer onChange implementieren, damit TanStack Form die Änderungen erhält -->
      <!-- onBlur-Event auf dem Feld abhören -->
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

Sie können also steuern, wann die Validierung durchgeführt wird, indem Sie das gewünschte Callback implementieren. Sie können sogar unterschiedliche Validierungen zu unterschiedlichen Zeitpunkten durchführen:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
      onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <!-- Wir müssen immer onChange implementieren, damit TanStack Form die Änderungen erhält -->
      <!-- onBlur-Event auf dem Feld abhören -->
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

Im obigen Beispiel validieren wir unterschiedliche Dinge für dasselbe Feld zu unterschiedlichen Zeitpunkten (bei jedem Tastendruck und beim Verlassen des Felds). Da `field.state.meta.errors` ein Array ist, werden alle relevanten Fehler zu einem bestimmten Zeitpunkt angezeigt. Sie können auch `field.state.meta.errorMap` verwenden, um Fehler basierend auf dem Zeitpunkt der Validierung (onChange, onBlur usw.) abzurufen. Mehr Informationen zur Anzeige von Fehlern finden Sie weiter unten.

## Fehler anzeigen

Sobald Sie Ihre Validierung eingerichtet haben, können Sie die Fehler aus einem Array für die Darstellung in Ihrer Benutzeroberfläche abbilden:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

Oder verwenden Sie die Eigenschaft `errorMap`, um auf den spezifischen Fehler zuzugreifen, nach dem Sie suchen:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
      <em role="alert" v-if="field.state.meta.errorMap['onChange']">{{
        field.state.meta.errorMap['onChange']
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

Es ist erwähnenswert, dass unser `errors`-Array und der `errorMap` den von den Validatoren zurückgegebenen Typen entsprechen. Das bedeutet:

```vue
<form.Field
  name="age"
  :validators="{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }"
>
  <template v-slot="{ field }">
      <!-- ... -->
      <!-- errorMap.onChange ist vom Typ `{isOldEnough: false} | undefined` -->
	  <!-- meta.errors ist vom Typ `Array<{isOldEnough: false} | undefined>` -->
      <em v-if="!field.state.meta.errorMap['onChange']?.isOldEnough">The user is not old enough</em>
  </template>
</form.Field>
```

## Validierung auf Feldebene vs. auf Formularebene

Wie oben gezeigt, akzeptiert jedes `<Field>` seine eigenen Validierungsregeln über die Callbacks `onChange`, `onBlur` usw. Es ist auch möglich, Validierungsregeln auf Formularebene (anstatt Feld für Feld) zu definieren, indem Sie ähnliche Callbacks an die Funktion `useForm()` übergeben.

Beispiel:

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    age: 0,
  },
  onSubmit: async ({ value }) => {
    console.log(value)
  },
  validators: {
    // Fügen Sie Validatoren dem Formular auf die gleiche Weise hinzu wie einem Feld
    onChange({ value }) {
      if (value.age < 13) {
        return 'Must be 13 or older to sign'
      }
      return undefined
    },
  },
})

// Abonnieren Sie die Error Map des Formulars, damit Aktualisierungen gerendert werden
// alternativ können Sie `form.Subscribe` verwenden
const formErrorMap = form.useStore((state) => state.errorMap)
</script>

<template>
  <!-- ... -->
  <div v-if="formErrorMap.onChange">
    <em role="alert">
      There was an error on the form: {{ formErrorMap.onChange }}
    </em>
  </div>
  <!-- ... -->
</template>
```

### Feldbezogene Fehler von den Validatoren des Formulars setzen

Sie können Fehler auf den Feldern von den Validatoren des Formulars aus setzen. Ein häufiger Anwendungsfall hierfür ist die Validierung aller Felder beim Absenden durch Aufrufen eines einzelnen API-Endpunkts im `onSubmitAsync`-Validator des Formulars.

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    age: 0,
    socials: [],
    details: {
      email: '',
    },
  },
  validators: {
    // Fügen Sie Validatoren dem Formular auf die gleiche Weise hinzu wie einem Feld
    onSubmitAsync({ value }) {
      // Validieren Sie den Wert auf dem Server
      const hasErrors = await verifyDataOnServer(value)
      if (hasErrors) {
        return {
          form: 'Invalid data', // Der Schlüssel `form` ist optional
          fields: {
            age: 'Must be 13 or older to sign',
            // Setzen Sie Fehler auf verschachtelte Felder mit dem Namen des Felds
            'socials[0].url': 'The provided URL does not exist',
            'details.email': 'An email is required',
          },
        }
      }

      return null
    },
  },
})
</script>

<template>
  <!-- ... -->
</template>
```

> Es ist erwähnenswert, dass, wenn Sie eine Formularvalidierungsfunktion haben, die einen Fehler zurückgibt, dieser Fehler durch die feldbezogene Validierung überschrieben werden kann.
>
> Das bedeutet:
>
> ```vue
> <script setup lang="ts">
> import { useForm } from '@tanstack/vue-form'
>
> const form = useForm({
>   defaultValues: {
>     age: 0,
>   },
>   validators: {
>     onChange: ({ value }) => {
>       return {
>         fields: {
>           age: value.age < 12 ? 'Too young!' : undefined,
>         },
>       }
>     },
>   },
> })
> </script>
>
> <template>
>   <!-- ... -->
>   <form.Field
>     name="age"
>     :validators="{
>       onChange: ({ value }) => (value % 2 === 0 ? 'Must be odd!' : undefined),
>     }"
>   >
>     <template v-slot="{ field }">
>       <!-- ... -->
>     </template>
>   </form.Field>
> </template>
> ```
>
> Wird nur `'Must be odd!'` anzeigen, selbst wenn der Fehler 'Too young!' von der Formularvalidierung zurückgegeben wird.

## Asynchrone funktionale Validierung

Während wir vermuten, dass die meisten Validierungen synchron sein werden, gibt es viele Fälle, in denen ein Netzwerkaufruf oder eine andere asynchrone Operation nützlich wäre, um dagegen zu validieren.

Dafür haben wir dedizierte Methoden wie `onChangeAsync`, `onBlurAsync` und andere, die zur Validierung verwendet werden können:

```vue
<script setup lang="ts">
// ...

const onChangeAge = async ({ value }) => {
  await new Promise((resolve) => setTimeout(resolve, 1000))
  return value < 13 ? 'You must be 13 to make an account' : undefined
}
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChangeAsync: onChangeAge,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @input="
          (e) =>
            field.handleChange((e.target as HTMLInputElement).valueAsNumber)
        "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

Synchrone und asynchrone Validierungen können nebeneinander existieren. Beispielsweise ist es möglich, sowohl `onBlur` als auch `onBlurAsync` für dasselbe Feld zu definieren:

```vue
<script setup lang="ts">
// ...

const onBlurAge = ({ value }) => (value < 0 ? 'Invalid value' : undefined)

const onBlurAgeAsync = async ({ value }) => {
  const currentAge = await fetchCurrentAgeOnProfile()
  return value < currentAge ? 'You can only increase the age' : undefined
}
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onBlur: onBlurAge,
      onBlurAsync: onBlurAgeAsync,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="
          (e) =>
            field.handleChange((e.target as HTMLInputElement).valueAsNumber)
        "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

Die synchrone Validierungsmethode (`onBlur`) wird zuerst ausgeführt, und die asynchrone Methode (`onBlurAsync`) wird nur ausgeführt, wenn die synchrone (`onBlur`) erfolgreich ist. Um dieses Verhalten zu ändern, setzen Sie die Option `asyncAlways` auf `true`, und die asynchrone Methode wird unabhängig vom Ergebnis der synchronen Methode ausgeführt.

### Integriertes Debouncing

Während asynchrone Aufrufe der richtige Weg sind, um gegen die Datenbank zu validieren, ist das Ausführen einer Netzwerkanfrage bei jedem Tastendruck eine gute Möglichkeit, Ihre Datenbank mit einem DDOS-Angriff zu überlasten.

Stattdessen ermöglichen wir eine einfache Methode zum Debouncen Ihrer `async`-Aufrufe, indem Sie eine einzelne Eigenschaft hinzufügen:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :async-debounce-ms="500"
    :validators="{
      onChangeAsync: async ({ value }) => {
        // ...
      },
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

Dies wird jeden asynchronen Aufruf mit einer Verzögerung von 500ms debouncen. Sie können diese Eigenschaft sogar für jede Validierungseigenschaft überschreiben:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :async-debounce-ms="500"
    :validators="{
      onChangeAsyncDebounceMs: 1500,
      onChangeAsync: async ({ value }) => {
        // ...
      },
      onBlurAsync: async ({ value }) => {
        // ...
      },
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

Dies führt `onChangeAsync` alle 1500ms aus, während `onBlurAsync` alle 500ms ausgeführt wird.

## Validierung mit Schema-Bibliotheken

Während Funktionen mehr Flexibilität und Anpassungsmöglichkeiten für Ihre Validierung bieten, können sie etwas umständlich sein. Um dies zu lösen, gibt es Bibliotheken, die schema-basierte Validierung bereitstellen, um Kurzschrift- und typstrikte Validierung erheblich einfacher zu machen. Sie können auch ein einzelnes Schema für Ihr gesamtes Formular definieren und es auf Formularebene übergeben, Fehler werden automatisch an die Felder weitergegeben.

### Standard-Schema-Bibliotheken

TanStack Form unterstützt nativ alle Bibliotheken, die der [Standard-Schema-Spezifikation](https://github.com/standard-schema/standard-schema) folgen, insbesondere:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_Hinweis:_ Stellen Sie sicher, dass Sie die neueste Version der Schema-Bibliotheken verwenden, da ältere Versionen möglicherweise noch kein Standard-Schema unterstützen.

> Die Validierung liefert Ihnen keine transformierten Werte. Weitere Informationen finden Sie unter [submission handling](./submission-handling.md).

Um Schemata dieser Bibliotheken zu verwenden, können Sie sie wie eine benutzerdefinierte Funktion an die `validators`-Props übergeben:

```vue
<script setup lang="ts">
import { z } from 'zod'
// ...

const form = useForm({
  // ...
})
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: z.number().gte(13, 'You must be 13 to make an account'),
    }"
  >
    <template v-slot="{ field }"></template
  ></form.Field>
</template>
```
