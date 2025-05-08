---
source-updated-at: '2025-05-08T07:42:29.000Z'
translation-updated-at: '2025-05-08T23:49:15.520Z'
id: basic-concepts
title: Grundkonzepte
---

Diese Seite führt die grundlegenden Konzepte und Begriffe ein, die in der `@tanstack/react-form`-Bibliothek verwendet werden. Das Vertrautmachen mit diesen Konzepten hilft Ihnen, die Bibliothek besser zu verstehen und effektiver damit zu arbeiten.

## Formular-Optionen

Sie können Optionen für Ihr Formular erstellen, um sie zwischen mehreren Formularen zu teilen, indem Sie die Funktion `formOptions` verwenden.

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

## Formular-Instanz

Eine Formular-Instanz ist ein Objekt, das ein individuelles Formular repräsentiert und Methoden sowie Eigenschaften für die Arbeit mit dem Formular bereitstellt. Sie erstellen eine Formular-Instanz mit dem Hook `useForm`, der von den Formular-Optionen bereitgestellt wird. Der Hook akzeptiert ein Objekt mit einer `onSubmit`-Funktion, die beim Absenden des Formulars aufgerufen wird.

```tsx
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // Mit den Formulardaten etwas tun
    console.log(value)
  },
})
```

Sie können auch eine Formular-Instanz ohne `formOptions` erstellen, indem Sie die eigenständige `useForm`-API verwenden:

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

Ein Feld repräsentiert ein einzelnes Formular-Eingabeelement, wie z.B. ein Textfeld oder eine Checkbox. Felder werden mit der Komponente `form.Field` erstellt, die von der Formular-Instanz bereitgestellt wird. Die Komponente akzeptiert eine `name`-Prop, die mit einem Schlüssel in den Standardwerten des Formulars übereinstimmen sollte. Sie akzeptiert auch eine `children`-Prop, eine Render-Prop-Funktion, die ein Feldobjekt als Argument erhält.

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

## Feld-Status

Jedes Feld hat seinen eigenen Status, der seinen aktuellen Wert, Validierungsstatus, Fehlermeldungen und weitere Metadaten enthält. Sie können den Status eines Felds über die Eigenschaft `field.state` abrufen.

Beispiel:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Es gibt drei Status in den Metadaten, die nützlich sein können, um die Interaktion des Benutzers mit einem Feld nachzuvollziehen:

- _"isTouched"_, nachdem der Benutzer in das Feld geklickt/getabt hat
- _"isPristine"_, bis der Benutzer den Feldwert ändert
- _"isDirty"_, nachdem der Feldwert geändert wurde

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Feld-Status](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## Verständnis von 'isDirty' in verschiedenen Bibliotheken

Nicht-persistenter `dirty`-Status

- **Bibliotheken**: React Hook Form (RHF), Formik, Final Form.
- **Verhalten**: Ein Feld ist 'dirty', wenn sein Wert vom Standardwert abweicht. Die Rückkehr zum Standardwert macht es wieder 'clean'.

Persistenter `dirty`-Status

- **Bibliotheken**: Angular Form, Vue FormKit.
- **Verhalten**: Ein Feld bleibt 'dirty', sobald es geändert wurde, selbst wenn es auf den Standardwert zurückgesetzt wird.

Wir haben uns für das Modell des persistenten 'dirty'-Status entschieden. Um auch einen nicht-persistenten 'dirty'-Status zu unterstützen, führen wir das Flag `isDefault` ein. Dieses Flag fungiert als Umkehrung des nicht-persistenten 'dirty'-Status.

```tsx
const { isTouched, isPristine, isDirty, isDefaultValue } = field.state.meta

// Die folgende Zeile reproduziert die Funktionalität des nicht-persistenten `dirty`-Status.
const nonPersistentIsDirty = !isDefaultValue
```

![Erweiterte Feld-Status](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states-extended.png)

## Feld-API

Die Feld-API ist ein Objekt, das an die Render-Prop-Funktion übergeben wird, wenn ein Feld erstellt wird. Sie bietet Methoden für die Arbeit mit dem Feldstatus.

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
      return value.includes('error') && '"error" ist im Vornamen nicht erlaubt'
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

Sie können ein Schema mit einer der Bibliotheken definieren, die die Spezifikation implementieren, und es einem Formular- oder Feld-Validator übergeben.

Unterstützte Bibliotheken sind:

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

`@tanstack/react-form` bietet verschiedene Möglichkeiten, um Änderungen am Formular- und Feldstatus zu abonnieren, insbesondere den Hook `useStore(form.store)` und die Komponente `form.Subscribe`. Diese Methoden ermöglichen es Ihnen, die Rendering-Performance Ihres Formulars zu optimieren, indem Komponenten nur bei Bedarf aktualisiert werden.

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

Es ist wichtig zu beachten, dass die `selector`-Prop des `useStore`-Hooks optional ist, aber dringend empfohlen wird, da das Weglassen unnötige Neu-Renderings verursacht.

```tsx
// Korrekte Verwendung
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
// Falsche Verwendung
const store = useStore(form.store)
```

Hinweis: Die Verwendung des `useField`-Hooks zur Erzielung von Reaktivität wird nicht empfohlen, da er gezielt innerhalb der `form.Field`-Komponente verwendet werden sollte. Stattdessen können Sie `useStore(form.store)` verwenden.

## Listener

`@tanstack/react-form` ermöglicht es Ihnen, auf bestimmte Trigger zu reagieren und diese zu "überwachen", um Nebenwirkungen auszulösen.

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

Weitere Informationen finden Sie unter [Listener](./listeners.md)

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

Wenn Sie `<button type="reset">` in Kombination mit `form.reset()` von TanStack Form verwenden, müssen Sie das standardmäßige HTML-Reset-Verhalten verhindern, um unerwartete Zurücksetzungen von Formularelementen (insbesondere `<select>`-Elementen) auf ihre initialen HTML-Werte zu vermeiden.
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

Dies sind die grundlegenden Konzepte und Begriffe, die in der `@tanstack/react-form`-Bibliothek verwendet werden. Das Verständnis dieser Konzepte hilft Ihnen, effektiver mit der Bibliothek zu arbeiten und komplexe Formulare mühelos zu erstellen.
