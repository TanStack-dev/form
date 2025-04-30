---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:57:53.775Z'
id: form-validation
title: Formularvalidierung
---

# Formular- und Feldvalidierung

Im Kern der Funktionalitäten von TanStack Form steht das Konzept der Validierung. TanStack Form macht die Validierung hochgradig anpassbar:

- Sie können steuern, wann die Validierung durchgeführt wird (bei Änderung, bei Eingabe, bei Verlassen des Felds, beim Absenden...)
- Validierungsregeln können auf Feldebene oder auf Formularebene definiert werden
- Die Validierung kann synchron oder asynchron erfolgen (z.B. als Ergebnis eines API-Aufrufs)

## Wann wird die Validierung durchgeführt?

Das liegt ganz bei Ihnen! Die Komponente `<form.Field />` akzeptiert einige Callbacks als Props wie `onChange` oder `onBlur`. Diese Callbacks erhalten den aktuellen Wert des Felds sowie das `fieldAPI`-Objekt, sodass Sie die Validierung durchführen können. Wenn Sie einen Validierungsfehler finden, geben Sie einfach die Fehlermeldung als String zurück, und sie wird in `field.state.meta.errors` verfügbar sein.

Hier ein Beispiel:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

Im obigen Beispiel wird die Validierung bei jedem Tastenanschlag durchgeführt (`onchange`). Wenn wir stattdessen die Validierung beim Verlassen des Felds durchführen wollten, würden wir den Code wie folgt ändern:

```svelte
<form.Field
  name="age"
  validators={{
    onBlur: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

Sie können also steuern, wann die Validierung durchgeführt wird, indem Sie den gewünschten Callback implementieren. Sie können sogar verschiedene Validierungen zu unterschiedlichen Zeitpunkten durchführen:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
    onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

Im obigen Beispiel validieren wir verschiedene Aspekte desselben Felds zu unterschiedlichen Zeitpunkten (bei jedem Tastenanschlag und beim Verlassen des Felds). Da `field.state.meta.errors` ein Array ist, werden alle relevanten Fehler zu einem bestimmten Zeitpunkt angezeigt. Sie können auch `field.state.meta.errorMap` verwenden, um Fehler basierend auf dem Zeitpunkt der Validierung (onChange, onBlur usw.) abzurufen. Mehr Informationen zur Anzeige von Fehlern finden Sie weiter unten.

## Fehler anzeigen

Sobald Sie Ihre Validierung eingerichtet haben, können Sie die Fehler aus einem Array für die Anzeige in Ihrer Benutzeroberfläche abbilden:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

Oder verwenden Sie die Eigenschaft `errorMap`, um auf den spezifischen Fehler zuzugreifen, den Sie suchen:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    {#if field.state.meta.errorMap['onChange']}
      <em role="alert">{field.state.meta.errorMap['onChange']}</em>
    {/if}
  {/snippet}
</form.Field>
```

Es ist erwähnenswert, dass unser `errors`-Array und der `errorMap` den von den Validatoren zurückgegebenen Typen entsprechen. Das bedeutet:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    <!-- errorMap.onChange ist vom Typ `{isOldEnough: false} | undefined` -->
    <!-- meta.errors ist vom Typ `Array<{isOldEnough: false} | undefined>` -->
    {#if field.state.meta.errorMap['onChange']?.isOldEnough}
        <em>The user is not old enough</em>
    {/if}
  {/snippet}
</form.Field>
```

## Validierung auf Feldebene vs. auf Formularebene

Wie oben gezeigt, akzeptiert jedes `<form.Field>` seine eigenen Validierungsregeln über die Callbacks `onChange`, `onBlur` usw. Es ist auch möglich, Validierungsregeln auf Formularebene (im Gegensatz zu Feld für Feld) zu definieren, indem Sie ähnliche Callbacks an den `createForm()`-Hook übergeben.

Beispiel:

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
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
  }))

  // Abonnieren Sie die Fehlerkarte des Formulars, damit Aktualisierungen gerendert werden
  // alternativ können Sie `form.Subscribe` verwenden
  const formErrorMap = form.useStore((state) => state.errorMap)
</script>

<div>
  <!-- ... -->
  {#if formErrorMap.current.onChange}
    <div>
      <em>There was an error on the form: {formErrorMap.current.onChange}</em>
    </div>
  {/if}
  <!-- ... -->
</div>
```

## Asynchrone funktionale Validierung

Während wir vermuten, dass die meisten Validierungen synchron sein werden, gibt es viele Fälle, in denen ein Netzwerkaufruf oder eine andere asynchrone Operation nützlich wäre.

Dafür haben wir dedizierte Methoden wie `onChangeAsync`, `onBlurAsync` und andere, die für die Validierung verwendet werden können:

```svelte
<form.Field
  name="age"
  validators={{
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value < 13 ? 'You must be 13 to make an account' : undefined
    },
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

Synchrone und asynchrone Validierungen können nebeneinander existieren. Beispielsweise ist es möglich, sowohl `onBlur` als auch `onBlurAsync` für dasselbe Feld zu definieren:

```svelte
<form.Field
  name="age"
  validators={{
    onBlur: ({ value }) => (value < 13 ? 'You must be at least 13' : undefined),
    onBlurAsync: async ({ value }) => {
      const currentAge = await fetchCurrentAgeOnProfile()
      return value < currentAge ? 'You can only increase the age' : undefined
    },
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

Die synchrone Validierungsmethode (`onBlur`) wird zuerst ausgeführt, und die asynchrone Methode (`onBlurAsync`) wird nur ausgeführt, wenn die synchrone erfolgreich ist. Um dieses Verhalten zu ändern, setzen Sie die Option `asyncAlways` auf `true`, und die asynchrone Methode wird unabhängig vom Ergebnis der synchronen Methode ausgeführt.

### Integriertes Debouncing

Während asynchrone Aufrufe der richtige Weg sind, um gegen die Datenbank zu validieren, ist ein Netzwerkaufruf bei jedem Tastenanschlag eine gute Möglichkeit, Ihre Datenbank mit einem DDOS-Angriff zu überlasten.

Stattdessen ermöglichen wir eine einfache Methode zum Debouncing Ihrer `async`-Aufrufe durch Hinzufügen einer einzigen Eigenschaft:

```svelte
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsync: async ({ value }) => {
      // ...
    },
  }}
>
  <!-- ... -->
</form.Field>
```

Dies wird jeden asynchronen Aufruf mit einer Verzögerung von 500ms debouncen. Sie können diese Eigenschaft sogar pro Validierungseigenschaft überschreiben:

```svelte
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsyncDebounceMs: 1500,
    onChangeAsync: async ({ value }) => {
      // ...
    },
    onBlurAsync: async ({ value }) => {
      // ...
    },
  }}
>
  <!-- ... -->
</form.Field>
```

> Dies führt `onChangeAsync` alle 1500ms aus, während `onBlurAsync` alle 500ms ausgeführt wird.

## Validierung mit Schema-Bibliotheken

Während Funktionen mehr Flexibilität und Anpassungsmöglichkeiten für Ihre Validierung bieten, können sie etwas umständlich sein. Um dies zu lösen, gibt es Bibliotheken, die schema-basierte Validierung bereitstellen, um Kurzschrift- und typstrikte Validierung wesentlich einfacher zu machen. Sie können auch ein einzelnes Schema für Ihr gesamtes Formular definieren und es auf Formularebene übergeben, Fehler werden automatisch an die Felder weitergegeben.

### Standard-Schema-Bibliotheken

TanStack Form unterstützt nativ alle Bibliotheken, die der [Standard-Schema-Spezifikation](https://github.com/standard-schema/standard-schema) folgen, insbesondere:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_Hinweis:_ Stellen Sie sicher, dass Sie die neueste Version der Schema-Bibliotheken verwenden, da ältere Versionen möglicherweise noch kein Standard-Schema unterstützen.

Um Schemata aus diesen Bibliotheken zu verwenden, können Sie sie wie eine benutzerdefinierte Funktion an die `validators`-Props übergeben:

```svelte
<script>
  import { z } from 'zod'

  // ...

  const form = createForm(() => ({
    // ...
  }))
</script>

<form.Field
  name="age"
  validators={{
    onChange: z.number().gte(13, 'You must be 13 to make an account'),
  }}
>
  <!-- ... -->
</form.Field>
```

Asynchrone Validierungen auf Formular- und Feldebene werden ebenfalls unterstützt:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: z.number().gte(13, 'You must be 13 to make an account'),
    onChangeAsyncDebounceMs: 500,
    onChangeAsync: z.number().refine(
      async (value) => {
        const currentAge = await fetchCurrentAgeOnProfile()
        return value >= currentAge
      },
      {
        message: 'You can only increase the age',
      },
    ),
  }}
>
  <!-- ... -->
</form.Field>
```

## Verhindern, dass ungültige Formulare abgesendet werden

Die Callbacks `onChange`, `onBlur` usw. werden auch beim Absenden des Formulars ausgeführt, und die Übermittlung wird blockiert, wenn das Formular ungültig ist.

Das Formularzustandsobjekt hat ein Flag `canSubmit`, das false ist, wenn ein Feld ungültig ist und das Formular berührt wurde (`canSubmit` ist true, bis das Formular berührt wurde, auch wenn einige Felder "technisch gesehen" ungültig sind basierend auf ihren `onChange`/`onBlur`-Props).

Sie können es über `form.Subscribe` abonnieren und den Wert verwenden, um beispielsweise den Absendebutton zu deaktivieren, wenn das Formular ungültig ist (in der Praxis sind deaktivierte Buttons nicht barrierefrei, verwenden Sie stattdessen `aria-disabled`).

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    /* ... */
  }))
</script>

<!-- ... -->

<!-- Dynamischer Absendebutton -->
<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
  children={(state) => (
    <button type="submit" disabled={!state().canSubmit}>
      {state().isSubmitting ? '...' : 'Submit'}
    </button>
  )}
>
  {#snippet children(state)}
    <button type="submit" disabled={!state.canSubmit}>
      {state.isSubmitting ? '...' : 'Submit'}
    </button>
  {/snippet}
</form.Subscribe>
```
