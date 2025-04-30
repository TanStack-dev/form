---
source-updated-at: '2025-04-16T08:45:06.000Z'
translation-updated-at: '2025-04-30T21:54:28.077Z'
id: basic-concepts
title: Grundkonzepte
---

Diese Seite führt die grundlegenden Konzepte und Begriffe ein, die in der `@tanstack/react-form`-Bibliothek verwendet werden. Wenn Sie sich mit diesen Konzepten vertraut machen, können Sie die Bibliothek besser verstehen und damit arbeiten.

## Formularoptionen

Sie können Optionen für Ihr Formular erstellen, damit es zwischen mehreren Formularen gemeinsam genutzt werden kann, indem Sie die Funktion `formOptions` verwenden.

Beispiel:

```tsx
interface User {
  firstName: string
  lastName: string
  hobbies: Array<string>
}
const defaultUser: User = { firstName: '', lastName: '', hobbies: [] }

const formOpts = formOptions({
  defaultValues: defaultUser,
})
```

## Formularinstanz

Eine Formularinstanz ist ein Objekt, das ein einzelnes Formular repräsentiert und Methoden und Eigenschaften zur Arbeit mit dem Formular bereitstellt. Sie erstellen eine Formularinstanz mit dem Hook `useForm`, der von den Formularoptionen bereitgestellt wird. Der Hook akzeptiert ein Objekt mit einer `onSubmit`-Funktion, die aufgerufen wird, wenn das Formular abgesendet wird.

```tsx
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // Mit den Formulardaten etwas tun
    console.log(value)
  },
})
```

Sie können auch eine Formularinstanz erstellen, ohne `formOptions` zu verwenden, indem Sie die eigenständige `useForm`-API nutzen:

```tsx
interface User {
  firstName: string
  lastName: string
  hobbies: Array<string>
}
const defaultUser: User = { firstName: '', lastName: '', hobbies: [] }

const form = useForm({
  defaultValues: defaultUser,
  onSubmit: async ({ value }) => {
    // Mit den Formulardaten etwas tun
    console.log(value)
  },
})
```

## Feld

Ein Feld (Field) repräsentiert ein einzelnes Formulareingabeelement, wie z.B. ein Textfeld oder eine Checkbox. Felder werden mit der Komponente `form.Field` erstellt, die von der Formularinstanz bereitgestellt wird. Die Komponente akzeptiert eine `name`-Prop, die mit einem Schlüssel in den Standardwerten des Formulars übereinstimmen sollte. Sie akzeptiert auch eine `children`-Prop, die eine Render-Prop-Funktion ist, die ein Feldobjekt als Argument erhält.

Beispiel:

```tsx
<form.Field
  name="firstName"
  children={(field) => (
    <>
      <input
        value={field.state.value}
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.value)}
      />
      <FieldInfo field={field} />
    </>
  )}
/>
```

## Feldzustand

Jedes Feld hat seinen eigenen Zustand (State), der seinen aktuellen Wert, Validierungsstatus, Fehlermeldungen und andere Metadaten enthält. Sie können den Zustand eines Felds über die Eigenschaft `field.state` abrufen.

Beispiel:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Es gibt drei Feldzustände, die nützlich sein können, um zu sehen, wie der Benutzer mit einem Feld interagiert: Ein Feld ist _"touched"_ (berührt), wenn der Benutzer hineinklickt/-tabuliert, _"pristine"_ (unverändert), bis der Benutzer den Wert ändert, und _"dirty"_ (verändert), nachdem der Wert geändert wurde. Sie können diese Zustände über die Flags `isTouched`, `isPristine` und `isDirty` überprüfen, wie unten gezeigt.

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Feldzustände](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

> **Wichtiger Hinweis für Benutzer von `React Hook Form`**: Das `isDirty`-Flag in `TanStack/form` unterscheidet sich von dem Flag mit demselben Namen in RHF.
> In RHF ist `isDirty = true`, wenn die Formularwerte von den ursprünglichen Werten abweichen. Wenn der Benutzer die Werte in einem Formular ändert und sie dann wieder ändert, sodass sie mit den Standardwerten des Formulars übereinstimmen, ist `isDirty` in RHF `false`, aber in `TanStack/form` `true`.
> Die Standardwerte sind sowohl auf Formular- als auch auf Feldebene in `TanStack/form` verfügbar (`form.options.defaultValues`, `field.options.defaultValue`), sodass Sie Ihre eigene `isDefaultValue()`-Hilfsfunktion schreiben können, wenn Sie das Verhalten von RHF nachbilden müssen.

## Feld-API

Die Feld-API (Field API) ist ein Objekt, das an die Render-Prop-Funktion übergeben wird, wenn ein Feld erstellt wird. Sie bietet Methoden zur Arbeit mit dem Zustand des Felds.

Beispiel:

```tsx
<input
  value={field.state.value}
  onBlur={field.handleBlur}
  onChange={(e) => field.handleChange(e.target.value)}
/>
```

## Validierung

`@tanstack/react-form` bietet sowohl synchrone als auch asynchrone Validierung von Haus aus. Validierungsfunktionen können der `form.Field`-Komponente über die `validators`-Prop übergeben werden.

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
      return value.includes('error') && '„error“ ist im Vornamen nicht erlaubt'
    },
  }}
  children={(field) => (
    <>
      <input
        value={field.state.value}
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.value)}
      />
      <FieldInfo field={field} />
    </>
  )}
/>
```

## Validierung mit Standard-Schema-Bibliotheken

Zusätzlich zu manuell erstellten Validierungsoptionen unterstützen wir auch die [Standard-Schema](https://github.com/standard-schema/standard-schema)-Spezifikation.

Sie können ein Schema mit einer der Bibliotheken definieren, die die Spezifikation implementieren, und es einem Formular- oder Feldvalidator übergeben.

Unterstützte Bibliotheken umfassen:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```tsx
import { z } from 'zod'

const userSchema = z.object({
  age: z
    .number()
    .gte(
      13,
      'Sie müssen mindestens 13 Jahre alt sein, um ein Konto zu erstellen',
    ),
})

function App() {
  const form = useForm({
    defaultValues: {
      age: 0,
    },
    validators: {
      onChange: userSchema,
    },
  })
  return (
    <div>
      <form.Field
        name="age"
        children={(field) => {
          return <>{/* ... */}</>
        }}
      />
    </div>
  )
}
```

## Reaktivität

`@tanstack/react-form` bietet verschiedene Möglichkeiten, um Änderungen am Formular- und Feldzustand zu abonnieren, insbesondere den Hook `useStore(form.store)` und die Komponente `form.Subscribe`. Diese Methoden ermöglichen es Ihnen, die Rendering-Performance Ihres Formulars zu optimieren, indem Komponenten nur bei Bedarf aktualisiert werden.

Beispiel:

```tsx
const firstName = useStore(form.store, (state) => state.values.firstName)
//...
<form.Subscribe
  selector={(state) => [state.canSubmit, state.isSubmitting]}
  children={([canSubmit, isSubmitting]) => (
    <button type="submit" disabled={!canSubmit}>
      {isSubmitting ? '...' : 'Absenden'}
    </button>
  )}
/>
```

Es ist wichtig zu beachten, dass die `selector`-Prop des `useStore`-Hooks optional ist, aber dringend empfohlen wird, da das Weglassen zu unnötigen Neu-Renderings führt.

```tsx
// Korrekte Verwendung
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
// Falsche Verwendung
const store = useStore(form.store)
```

Hinweis: Die Verwendung des `useField`-Hooks zur Erzielung von Reaktivität wird nicht empfohlen, da er gezielt innerhalb der `form.Field`-Komponente verwendet werden soll. Stattdessen können Sie `useStore(form.store)` verwenden.

## Listener

`@tanstack/react-form` ermöglicht es Ihnen, auf bestimmte Trigger zu reagieren und sie zu "überwachen", um Nebenwirkungen auszulösen.

Beispiel:

```tsx
<form.Field
  name="country"
  listeners={{
    onChange: ({ value }) => {
      console.log(`Land geändert zu: ${value}, Bundesland wird zurückgesetzt`)
      form.setFieldValue('province', '')
    },
  }}
/>
```

Weitere Informationen finden Sie unter [Listener](./listeners.md).

## Array-Felder

Array-Felder (Array Fields) ermöglichen es Ihnen, eine Liste von Werten innerhalb eines Formulars zu verwalten, wie z.B. eine Liste von Hobbys. Sie können ein Array-Feld mit der `form.Field`-Komponente und der `mode="array"`-Prop erstellen.

Bei der Arbeit mit Array-Feldern können Sie die Methoden `pushValue`, `removeValue`, `swapValues` und `moveValue` verwenden, um Werte im Array hinzuzufügen, zu entfernen und zu vertauschen.

Beispiel:

```tsx
<form.Field
  name="hobbies"
  mode="array"
  children={(hobbiesField) => (
    <div>
      Hobbys
      <div>
        {!hobbiesField.state.value.length
          ? 'Keine Hobbys gefunden.'
          : hobbiesField.state.value.map((_, i) => (
              <div key={i}>
                <form.Field
                  name={`hobbies[${i}].name`}
                  children={(field) => {
                    return (
                      <div>
                        <label htmlFor={field.name}>Name:</label>
                        <input
                          id={field.name}
                          name={field.name}
                          value={field.state.value}
                          onBlur={field.handleBlur}
                          onChange={(e) => field.handleChange(e.target.value)}
                        />
                        <button
                          type="button"
                          onClick={() => hobbiesField.removeValue(i)}
                        >
                          X
                        </button>
                        <FieldInfo field={field} />
                      </div>
                    )
                  }}
                />
                <form.Field
                  name={`hobbies[${i}].description`}
                  children={(field) => {
                    return (
                      <div>
                        <label htmlFor={field.name}>Beschreibung:</label>
                        <input
                          id={field.name}
                          name={field.name}
                          value={field.state.value}
                          onBlur={field.handleBlur}
                          onChange={(e) => field.handleChange(e.target.value)}
                        />
                        <FieldInfo field={field} />
                      </div>
                    )
                  }}
                />
              </div>
            ))}
      </div>
      <button
        type="button"
        onClick={() =>
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
  )}
/>
```

## Reset-Buttons

Wenn Sie `<button type="reset">` in Verbindung mit `form.reset()` von TanStack Form verwenden, müssen Sie das standardmäßige HTML-Reset-Verhalten verhindern, um unerwartete Zurücksetzungen von Formularelementen (insbesondere `<select>`-Elementen) auf ihre anfänglichen HTML-Werte zu vermeiden.
Verwenden Sie `event.preventDefault()` im `onClick`-Handler des Buttons, um das native Formular-Reset zu verhindern.

Beispiel:

```tsx
<button
  type="reset"
  onClick={(event) => {
    event.preventDefault()
    form.reset()
  }}
>
  Zurücksetzen
</button>
```

Alternativ können Sie `<button type="button">` verwenden, um das native HTML-Reset zu verhindern.

```tsx
<button
  type="button"
  onClick={() => {
    form.reset()
  }}
>
  Zurücksetzen
</button>
```

Dies sind die grundlegenden Konzepte und Begriffe, die in der `@tanstack/react-form`-Bibliothek verwendet werden. Wenn Sie diese Konzepte verstehen, können Sie effektiver mit der Bibliothek arbeiten und komplexe Formulare mühelos erstellen.
