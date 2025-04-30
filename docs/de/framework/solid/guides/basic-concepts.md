---
source-updated-at: '2025-03-14T21:54:05.000Z'
translation-updated-at: '2025-04-30T21:55:56.272Z'
id: basic-concepts
title: Grundkonzepte
---

Diese Seite führt in die grundlegenden Konzepte und Begriffe der `@tanstack/solid-form`-Bibliothek ein. Das Vertrautmachen mit diesen Konzepten hilft Ihnen, die Bibliothek besser zu verstehen und effektiver damit zu arbeiten.

## Formularoptionen

Sie können Optionen für Ihr Formular erstellen, die zwischen mehreren Formularen geteilt werden können, indem Sie die Funktion `formOptions` verwenden.

Beispiel:

```tsx
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## Formularinstanz

Eine Formularinstanz ist ein Objekt, das ein einzelnes Formular repräsentiert und Methoden sowie Eigenschaften für die Arbeit mit dem Formular bereitstellt. Sie erstellen eine Formularinstanz mit dem Hook `createForm`, der von den Formularoptionen bereitgestellt wird. Der Hook akzeptiert ein Objekt mit einer `onSubmit`-Funktion, die beim Absenden des Formulars aufgerufen wird.

```tsx
const form = createForm(() => ({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // Mit den Formulardaten etwas tun
    console.log(value)
  },
}))
```

Sie können auch eine Formularinstanz ohne `formOptions` erstellen, indem Sie die eigenständige `createForm`-API verwenden:

```tsx
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

## Feld

Ein Feld (Field) repräsentiert ein einzelnes Formulareingabeelement, wie z.B. ein Textfeld oder eine Checkbox. Felder werden mit der Komponente `form.Field` erstellt, die von der Formularinstanz bereitgestellt wird. Die Komponente akzeptiert eine `name`-Prop, die mit einem Schlüssel in den Standardwerten des Formulars übereinstimmen sollte. Sie akzeptiert auch eine `children`-Prop, eine Render-Prop-Funktion, die ein Feldobjekt als Argument erhält.

Beispiel:

```tsx
<form.Field
  name="firstName"
  children={(field) => (
    <input
      name={field().name}
      value={field().state.value}
      onBlur={field().handleBlur}
      onInput={(e) => field().handleChange(e.target.value)}
    />
  )}
/>
```

## Feldzustand

Jedes Feld hat seinen eigenen Zustand (State), der seinen aktuellen Wert, Validierungsstatus, Fehlermeldungen und andere Metadaten enthält. Sie können den Zustand eines Felds über die Eigenschaft `field().state` abrufen.

Beispiel:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field().state
```

Es gibt drei Feldzustände, die sehr nützlich sind, um die Interaktion des Benutzers mit einem Feld nachzuvollziehen. Ein Feld ist _"berührt"_ (touched), wenn der Benutzer hineinklickt/-tabuliert, _"unverändert"_ (pristine), bis der Benutzer den Wert ändert, und _"verändert"_ (dirty), nachdem der Wert geändert wurde. Sie können diese Zustände über die Flags `isTouched`, `isPristine` und `isDirty` prüfen, wie unten gezeigt.

```tsx
const { isTouched, isPristine, isDirty } = field().state.meta
```

![Feldzustände](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## Feld-API

Die Feld-API ist ein Objekt, das an die Render-Prop-Funktion übergeben wird, wenn ein Feld erstellt wird. Sie bietet Methoden für die Arbeit mit dem Zustand des Felds.

Beispiel:

```tsx
<input
  name={field().name}
  value={field().state.value}
  onBlur={field().handleBlur}
  onInput={(e) => field().handleChange(e.target.value)}
/>
```

## Validierung

`@tanstack/solid-form` bietet sowohl synchrone als auch asynchrone Validierung von Haus aus. Validierungsfunktionen können der `form.Field`-Komponente über die `validators`-Prop übergeben werden.

Beispiel:

```tsx
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
      return value.includes('error') && 'Kein "error" im Vornamen erlaubt'
    },
  }}
  children={(field) => (
    <>
      <input
        name={field().name}
        value={field().state.value}
        onBlur={field().handleBlur}
        onInput={(e) => field().handleChange(e.target.value)}
      />
      <p>{field().state.meta.errors[0]}</p>
    </>
  )}
/>
```

## Validierung mit Standard-Schema-Bibliotheken

Zusätzlich zu manuell erstellten Validierungsoptionen unterstützen wir auch die [Standard-Schema](https://github.com/standard-schema/standard-schema)-Spezifikation.

Sie können ein Schema mit einer der Bibliotheken definieren, die die Spezifikation implementieren, und es einem Formular- oder Feldvalidator übergeben.

Unterstützte Bibliotheken sind:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```tsx
import { z } from 'zod'

// ...
;<form.Field
  name="firstName"
  validators={{
    onChange: z
      .string()
      .min(3, 'Der Vorname muss mindestens 3 Zeichen lang sein'),
    onChangeAsyncDebounceMs: 500,
    onChangeAsync: z.string().refine(
      async (value) => {
        await new Promise((resolve) => setTimeout(resolve, 1000))
        return !value.includes('error')
      },
      {
        message: 'Kein "error" im Vornamen erlaubt',
      },
    ),
  }}
  children={(field) => (
    <>
      <input
        name={field().name}
        value={field().state.value}
        onBlur={field().handleBlur}
        onInput={(e) => field().handleChange(e.target.value)}
      />
      <p>{field().state.meta.errors[0]}</p>
    </>
  )}
/>
```

## Reaktivität

`@tanstack/solid-form` bietet verschiedene Möglichkeiten, um Änderungen am Formular- und Feldzustand zu abonnieren, insbesondere den Hook `form.useStore` und die Komponente `form.Subscribe`. Diese Methoden ermöglichen es Ihnen, die Rendering-Performance Ihres Formulars zu optimieren, indem Komponenten nur bei Bedarf aktualisiert werden.

Beispiel:

```tsx
const firstName = form.useStore((state) => state.values.firstName)
//...
<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
  children={(state) => (
    <button type="submit" disabled={!state().canSubmit}>
      {state().isSubmitting ? '...' : 'Absenden'}
    </button>
  )}
/>
```

## Array-Felder

Array-Felder ermöglichen es Ihnen, eine Liste von Werten innerhalb eines Formulars zu verwalten, wie z.B. eine Liste von Hobbys. Sie können ein Array-Feld mit der `form.Field`-Komponente und der `mode="array"`-Prop erstellen.

Bei der Arbeit mit Array-Feldern können Sie die Methoden `pushValue`, `removeValue`, `swapValues` und `moveValue` verwenden, um Werte im Array hinzuzufügen, zu entfernen oder zu vertauschen.

Beispiel:

```tsx
<form.Field
  name="hobbies"
  mode="array"
  children={(hobbiesField) => (
    <div>
      Hobbys
      <div>
        <Show
          when={hobbiesField().state.value.length > 0}
          fallback={'Keine Hobbys gefunden.'}
        >
          <Index each={hobbiesField().state.value}>
            {(_, i) => (
              <div>
                <form.Field
                  name={`hobbies[${i}].name`}
                  children={(field) => (
                    <div>
                      <label for={field().name}>Name:</label>
                      <input
                        id={field().name}
                        name={field().name}
                        value={field().state.value}
                        onBlur={field().handleBlur}
                        onInput={(e) => field().handleChange(e.target.value)}
                      />
                      <button
                        type="button"
                        onClick={() => hobbiesField().removeValue(i)}
                      >
                        X
                      </button>
                    </div>
                  )}
                />
                <form.Field
                  name={`hobbies[${i}].description`}
                  children={(field) => {
                    return (
                      <div>
                        <label for={field().name}>Beschreibung:</label>
                        <input
                          id={field().name}
                          name={field().name}
                          value={field().state.value}
                          onBlur={field().handleBlur}
                          onInput={(e) => field().handleChange(e.target.value)}
                        />
                      </div>
                    )
                  }}
                />
              </div>
            )}
          </Index>
        </Show>
      </div>
      <button
        type="button"
        onClick={() =>
          hobbiesField().pushValue({
            name: '',
            description: '',
            yearsOfExperience: 0,
          })
        }
      >
        Hobby hinzufügen
      </button>
    </div>
  )}
/>
```

Dies sind die grundlegenden Konzepte und Begriffe, die in der `@tanstack/solid-form`-Bibliothek verwendet werden. Das Verständnis dieser Konzepte hilft Ihnen, effektiver mit der Bibliothek zu arbeiten und komplexe Formulare mit Leichtigkeit zu erstellen.
