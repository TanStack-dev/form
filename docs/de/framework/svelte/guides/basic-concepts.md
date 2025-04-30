---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:57:07.437Z'
id: basic-concepts
title: Grundkonzepte
---

Diese Seite führt in die grundlegenden Konzepte und Begriffe der `@tanstack/svelte-form`-Bibliothek ein. Wenn Sie sich mit diesen Konzepten vertraut machen, können Sie die Bibliothek besser verstehen und effektiver nutzen.

## Formular-Optionen (Form Options)

Sie können Optionen für Ihr Formular erstellen, um sie zwischen mehreren Formularen zu teilen, indem Sie die Funktion `formOptions` verwenden.

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

## Formular-Instanz (Form Instance)

Eine Formular-Instanz ist ein Objekt, das ein einzelnes Formular repräsentiert und Methoden sowie Eigenschaften zur Arbeit mit dem Formular bereitstellt. Sie erstellen eine Formular-Instanz mit der Funktion `createForm`. Die Funktion akzeptiert ein Objekt mit einer `onSubmit`-Funktion, die beim Absenden des Formulars aufgerufen wird.

```ts
const form = createForm(() => ({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // Mit den Formulardaten etwas tun
    console.log(value)
  },
}))
```

Sie können auch eine Formular-Instanz ohne `formOptions` erstellen:

```ts
const form = createForm<Person>(() => ({
  onSubmit: async ({ value }) => {
    // Mit den Formulardaten etwas tun
    console.log(value)
  },
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  },
}))
```

## Feld (Field)

Ein Feld (Field) repräsentiert ein einzelnes Formulareingabeelement, wie z.B. ein Textfeld oder eine Checkbox. Felder werden mit der `form.Field`-Komponente erstellt, die von der Formular-Instanz bereitgestellt wird. Die Komponente akzeptiert eine `name`-Prop, die mit einem Schlüssel in den Standardwerten des Formulars übereinstimmen sollte. Sie akzeptiert auch eine `children`-Prop, die eine Render-Prop-Funktion ist, die ein Feld-Objekt als Argument erhält.

Beispiel:

```svelte
<form.Field name="firstName">
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onblur={field.handleBlur}
      oninput={(e) => field.handleChange(e.target.value)}
    />
  {/snippet}
</form.Field>
```

## Feld-Status (Field State)

Jedes Feld hat seinen eigenen Status, der seinen aktuellen Wert, Validierungsstatus, Fehlermeldungen und andere Metadaten enthält. Sie können den Status eines Felds über die Eigenschaft `field.state` abrufen.

Beispiel:

```ts
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Es gibt drei Feld-Status, die sehr nützlich sein können, um zu sehen, wie der Benutzer mit einem Feld interagiert. Ein Feld ist _"berührt" (touched)_, wenn der Benutzer hineinklickt oder -tabuliert, _"unverändert" (pristine)_, bis der Benutzer den Wert ändert, und _"verändert" (dirty)_, nachdem der Wert geändert wurde. Sie können diese Status über die Flags `isTouched`, `isPristine` und `isDirty` prüfen, wie unten gezeigt.

```ts
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Feld-Status](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## Feld-API (Field API)

Die Feld-API ist ein Objekt, das an die Render-Prop-Funktion übergeben wird, wenn ein Feld erstellt wird. Es bietet Methoden zur Arbeit mit dem Status des Felds.

Beispiel:

```svelte
<input
  name={field.name}
  value={field.state.value}
  onblur={field.handleBlur}
  oninput={(e) => field.handleChange(e.target.value)}
/>
```

## Validierung (Validation)

`@tanstack/svelte-form` bietet sowohl synchrone als auch asynchrone Validierung von Haus aus. Validierungsfunktionen können der `form.Field`-Komponente über die `validators`-Prop übergeben werden.

Beispiel:

```svelte
<form.Field
  name="firstName"
  validators={{
    onChange: ({ value }) =>
      !value
        ? 'Ein Vorname ist erforderlich'
        : value.length < 3
          ? 'Der Vorname muss mindestens 3 Zeichen lang sein'
          : undefined,
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && 'Das Wort "error" ist im Vornamen nicht erlaubt'
    },
  }}
>
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onBlur={field.handleBlur}
      onInput={(e) => field.handleChange(e.target.value)}
    />
    <p>{field.state.meta.errors[0]}</p>
  {/snippet}
</form.Field>
```

## Validierung mit Standard-Schema-Bibliotheken (Validation with Standard Schema Libraries)

Zusätzlich zu manuell erstellten Validierungsoptionen unterstützen wir auch die [Standard-Schema](https://github.com/standard-schema/standard-schema)-Spezifikation.

Sie können ein Schema mit einer der Bibliotheken erstellen, die die Spezifikation implementieren, und es einem Formular- oder Feld-Validator übergeben.

Unterstützte Bibliotheken sind:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```svelte
<script>
  import { z } from 'zod'

  // ...
</script>

<form.Field
  name="firstName"
  validators={{
    onChange: z.string().min(3, 'Der Vorname muss mindestens 3 Zeichen lang sein'),
    onChangeAsyncDebounceMs: 500,
    onChangeAsync: z.string().refine(
      async (value) => {
        await new Promise((resolve) => setTimeout(resolve, 1000))
        return !value.includes('error')
      },
      {
        message: 'Das Wort "error" ist im Vornamen nicht erlaubt',
      },
    ),
  }}
>
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onBlur={field.handleBlur}
      onInput={(e) => field.handleChange(e.target.value)}
    />
    <p>{field.state.meta.errors[0]}</p>
  {/snippet}
</form.Field>
```

## Reaktivität (Reactivity)

`@tanstack/svelte-form` bietet verschiedene Möglichkeiten, um Änderungen am Formular- und Feldstatus zu abonnieren, insbesondere den `form.useStore`-Hook und die `form.Subscribe`-Komponente. Diese Methoden ermöglichen es Ihnen, die Rendering-Performance Ihres Formulars zu optimieren, indem Komponenten nur bei Bedarf aktualisiert werden.

Beispiel:

```svelte
<script>
  //...
  const firstName = form.useStore((state) => state.values.firstName)
</script>

<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
>
  {#snippet children(state)}
    <button type="submit" disabled={!state.canSubmit}>
      {state.isSubmitting ? '...' : 'Absenden'}
    </button>
  {/snippet}
</form.Subscribe>
```

## Array-Felder (Array Fields)

Array-Felder ermöglichen es Ihnen, eine Liste von Werten innerhalb eines Formulars zu verwalten, wie z.B. eine Liste von Hobbys. Sie können ein Array-Feld mit der `form.Field`-Komponente und der `mode="array"`-Prop erstellen.

Bei der Arbeit mit Array-Feldern können Sie die Methoden `pushValue`, `removeValue`, `swapValues` und `moveValue` verwenden, um Werte im Array hinzuzufügen, zu entfernen oder zu vertauschen.

Beispiel:

```svelte
<form.Field name="hobbies" mode="array">
  {#snippet children(hobbiesField)}
    <div>
      Hobbys
      <div>
        {#each hobbiesField.state.value as _, i}
            <div>
              <form.Field name={`hobbies[${i}].name`}>
                {#snippet children(field)}
                  <div>
                    <label for={field.name}>Name:</label>
                    <input
                      id={field.name}
                      name={field.name}
                      value={field.state.value}
                      onblur={field.handleBlur}
                      onchange={(e) => field.handleChange(e.target.value)}
                    />
                    <button
                      type="button"
                      onclick={() => hobbiesField.removeValue(i)}
                    >
                      X
                    </button>
                  </div>
                {/snippet}
              </form.Field>
              <form.Field name={`hobbies[${i}].description`}>
                {#snippet children(field)}
                    <div>
                      <label for={field.name}>Beschreibung:</label>
                      <input
                        id={field.name}
                        name={field.name}
                        value={field.state.value}
                        onblur={field.handleBlur}
                        onchange={(e) => field.handleChange(e.target.value)}
                      />
                    </div>
                {/snippet}
              </form.Field>
            </div>
          {:else}
            Keine Hobbys gefunden.
          {/each}
      </div>
      <button
        type="button"
        onclick={() =>
          hobbiesField.pushValue({
            name: '',
            description: '',
            yearsOfExperience: 0,
          })
        }
      >
        Hobby hinzufügen
      </button>
    </div>
  {/snippet}
</form.Field>
```

Dies sind die grundlegenden Konzepte und Begriffe, die in der `@tanstack/svelte-form`-Bibliothek verwendet werden. Wenn Sie diese Konzepte verstehen, können Sie effektiver mit der Bibliothek arbeiten und komplexe Formulare mühelos erstellen.
