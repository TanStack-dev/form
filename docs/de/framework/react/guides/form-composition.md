---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:55:14.144Z'
id: form-composition
title: Formularzusammensetzung
---

# Form Composition (Formularzusammensetzung)

Eine häufige Kritik an TanStack Form ist seine ausführliche Standardimplementierung. Während dies _für Lernzwecke_ nützlich sein kann – um das Verständnis unserer APIs zu fördern – ist es für Produktionsumgebungen nicht ideal.

Daher bieten wir, obwohl `form.Field` die leistungsstärkste und flexibelste Nutzung von TanStack Form ermöglicht, APIs an, die es umschließen und Ihren Anwendungscode weniger wortreich machen.

## Benutzerdefinierte Form-Hooks

Die leistungsfähigste Methode zur Zusammensetzung von Formularen ist die Erstellung benutzerdefinierter Form-Hooks. Dies ermöglicht die Erstellung eines Form-Hooks, der auf die Bedürfnisse Ihrer Anwendung zugeschnitten ist, einschließlich vorgebundener benutzerdefinierter UI-Komponenten und mehr.

In seiner grundlegendsten Form ist `createFormHook` eine Funktion, die einen `fieldContext` und `formContext` entgegennimmt und einen `useAppForm`-Hook zurückgibt.

> Dieser unangepasste `useAppForm`-Hook ist identisch zu `useForm`, aber das wird sich schnell ändern, sobald wir weitere Optionen zu `createFormHook` hinzufügen.

```tsx
import { createFormHookContexts, createFormHook } from '@tanstack/react-form'

// exportiere useFieldContext für die Verwendung in Ihren benutzerdefinierten Komponenten
export const { fieldContext, formContext, useFieldContext } =
  createFormHookContexts()

const { useAppForm } = createFormHook({
  fieldContext,
  formContext,
  // Wir werden später mehr über diese Optionen lernen
  fieldComponents: {},
  formComponents: {},
})

function App() {
  const form = useAppForm({
    // Unterstützt alle useForm-Optionen
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return <form.Field /> // ...
}
```

### Vorgebundene Feldkomponenten

Sobald dieses Grundgerüst vorhanden ist, können Sie beginnen, benutzerdefinierte Feld- und Formularkomponenten zu Ihrem Form-Hook hinzuzufügen.

> Hinweis: Der `useFieldContext` muss derselbe sein, der aus Ihrem benutzerdefinierten Form-Kontext exportiert wurde

```tsx
import { useFieldContext } from './form-context.tsx'

export function TextField({ label }: { label: string }) {
  // Das `Field` leitet ab, dass es einen `value`-Typ von `string` haben sollte
  const field = useFieldContext<string>()
  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}
```

Anschließend können Sie diese Komponente mit Ihrem Form-Hook registrieren.

```tsx
import { TextField } from './text-field.tsx'

const { useAppForm } = createFormHook({
  fieldContext,
  formContext,
  fieldComponents: {
    TextField,
  },
  formComponents: {},
})
```

Und in Ihrem Formular verwenden:

```tsx
function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return (
    // Beachten Sie `AppField` anstelle von `Field`; `AppField` stellt den erforderlichen Kontext bereit
    <form.AppField
      name="firstName"
      children={(field) => <field.TextField label="First Name" />}
    />
  )
}
```

Dies ermöglicht nicht nur die Wiederverwendung der UI Ihrer gemeinsamen Komponente, sondern behält auch die Typsicherheit bei, die Sie von TanStack Form erwarten: Tippfehler im `name` führen zu einem TypeScript-Fehler.

#### Ein Hinweis zur Performance

Während Context ein wertvolles Werkzeug im React-Ökosystem ist, gibt es berechtigte Bedenken vieler Nutzer, dass die Bereitstellung eines reaktiven Werts über einen Context unnötige Neu-Renderings verursacht.

> Mit dieser Performance-Sorge nicht vertraut? [Mark Eriksons Blogpost, der erklärt, warum Redux viele dieser Probleme löst](https://blog.isquaredsoftware.com/2021/01/context-redux-differences/), ist ein guter Ausgangspunkt.

Obwohl dies eine berechtigte Sorge ist, stellt es für TanStack Form kein Problem dar; die über Context bereitgestellten Werte sind nicht selbst reaktiv, sondern statische Klasseninstanzen mit reaktiven Eigenschaften ([unter Verwendung von TanStack Store als unsere Signals-Implementierung](https://tanstack.com/store)).

### Vorgebundene Formularkomponenten

Während `form.AppField` viele Probleme mit Field-Boilerplate und Wiederverwendbarkeit löst, löst es nicht das Problem der _Formular_-Boilerplate und Wiederverwendbarkeit.

Insbesondere die Möglichkeit, Instanzen von `form.Subscribe` für beispielsweise einen reaktiven Formular-Submit-Button zu teilen, ist ein häufiger Anwendungsfall.

```tsx
function SubscribeButton({ label }: { label: string }) {
  const form = useFormContext()
  return (
    <form.Subscribe selector={(state) => state.isSubmitting}>
      {(isSubmitting) => <button disabled={isSubmitting}>{label}</button>}
    </form.Subscribe>
  )
}

const { useAppForm, withForm } = createFormHook({
  fieldComponents: {},
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return (
    <form.AppForm>
      // Beachten Sie die `AppForm`-Komponenten-Wrapper; `AppForm` stellt den
      erforderlichen Kontext bereit
      <form.SubscribeButton label="Submit" />
    </form.AppForm>
  )
}
```

## Große Formulare in kleinere Teile aufteilen

Manchmal werden Formulare sehr groß; das passiert einfach. Obwohl TanStack Form große Formulare gut unterstützt, ist es nie angenehm, mit Dateien zu arbeiten, die hunderte oder tausende Codezeilen lang sind.

Um dies zu lösen, unterstützen wir die Aufteilung von Formularen in kleinere Teile mithilfe der `withForm` Higher-Order-Komponente.

```tsx
const { useAppForm, withForm } = createFormHook({
  fieldComponents: {
    TextField,
  },
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

const ChildForm = withForm({
  // Diese Werte werden nur zur Typüberprüfung verwendet und nicht zur Laufzeit
  // Dies ermöglicht es, `...formOpts` aus `formOptions` zu übernehmen, ohne die Optionen neu deklarieren zu müssen
  defaultValues: {
    firstName: 'John',
    lastName: 'Doe',
  },
  // Optional, fügt dem `render`-Funktion zusätzlich zu `form` Props hinzu
  props: {
    // Diese Props werden auch als Standardwerte für die `render`-Funktion gesetzt
    title: 'Child Form',
  },
  render: function Render({ form, title }) {
    return (
      <div>
        <p>{title}</p>
        <form.AppField
          name="firstName"
          children={(field) => <field.TextField label="First Name" />}
        />
        <form.AppForm>
          <form.SubscribeButton label="Submit" />
        </form.AppForm>
      </div>
    )
  },
})

function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return <ChildForm form={form} title={'Testing'} />
}
```

### `withForm` FAQ

> Warum eine Higher-Order-Komponente statt eines Hooks?

Obwohl Hooks die Zukunft von React sind, sind Higher-Order-Komponenten nach wie vor ein mächtiges Werkzeug für die Komposition. Insbesondere ermöglicht die API von `useForm` eine starke Typsicherheit, ohne dass Benutzer Generics übergeben müssen.

> Warum erhalte ich ESLint-Fehler zu Hooks in `render`?

ESLint sucht nach Hooks auf der obersten Ebene einer Funktion, und `render` wird möglicherweise nicht als Top-Level-Komponente erkannt, abhängig davon, wie Sie es definiert haben.

```tsx
// Dies führt zu ESLint-Fehlern bei der Hooks-Verwendung
const ChildForm = withForm({
  // ...
  render: ({ form, title }) => {
    // ...
  },
})
```

```tsx
// Dies funktioniert einwandfrei
const ChildForm = withForm({
  // ...
  render: function Render({ form, title }) {
    // ...
  },
})
```

## Tree-Shaking von Formular- und Feldkomponenten

Während die obigen Beispiele ideal für den Einstieg sind, sind sie nicht optimal für bestimmte Anwendungsfälle, in denen Sie möglicherweise hunderte von Formular- und Feldkomponenten haben.
Insbesondere möchten Sie möglicherweise nicht alle Ihre Formular- und Feldkomponenten im Bundle jeder Datei enthalten, die Ihren Form-Hook verwendet.

Um dies zu lösen, können Sie die `createFormHook`-TanStack-API mit den React-Komponenten `lazy` und `Suspense` kombinieren:

```typescript
// src/hooks/form-context.ts
import { createFormHookContexts } from '@tanstack/react-form'

export const { fieldContext, useFieldContext, formContext, useFormContext } =
  createFormHookContexts()
```

```tsx
// src/components/text-field.tsx
import { useFieldContext } from '../hooks/form-context.tsx'

export default function TextField({ label }: { label: string }) {
  const field = useFieldContext<string>()

  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}
```

```tsx
// src/hooks/form.ts
import { lazy } from 'react'
import { createFormHook } from '@tanstack/react-form'

const TextField = lazy(() => import('../components/text-fields.tsx'))

const { useAppForm, withForm } = createFormHook({
  fieldContext,
  formContext,
  fieldComponents: {
    TextField,
  },
  formComponents: {},
})
```

```tsx
// src/App.tsx
import { Suspense } from 'react'
import { PeoplePage } from './features/people/page.tsx'

export default function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <PeopleForm />
    </Suspense>
  )
}
```

Dies zeigt den Suspense-Fallback, während die `TextField`-Komponente geladen wird, und rendert anschließend das Formular, sobald es geladen ist.

## Alles zusammenfügen

Nachdem wir die Grundlagen der Erstellung benutzerdefinierter Form-Hooks behandelt haben, lassen Sie uns alles in einem einzigen Beispiel zusammenfassen.

```tsx
// /src/hooks/form.ts, zur Verwendung in der gesamten App
const { fieldContext, useFieldContext, formContext, useFormContext } =
  createFormHookContexts()

function TextField({ label }: { label: string }) {
  const field = useFieldContext<string>()
  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}

function SubscribeButton({ label }: { label: string }) {
  const form = useFormContext()
  return (
    <form.Subscribe selector={(state) => state.isSubmitting}>
      {(isSubmitting) => <button disabled={isSubmitting}>{label}</button>}
    </form.Subscribe>
  )
}

const { useAppForm, withForm } = createFormHook({
  fieldComponents: {
    TextField,
  },
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

// /src/features/people/shared-form.ts, zur Verwendung in `people`-Features
const formOpts = formOptions({
  defaultValues: {
    firstName: 'John',
    lastName: 'Doe',
  },
})

// /src/features/people/nested-form.ts, zur Verwendung in der `people`-Seite
const ChildForm = withForm({
  ...formOpts,
  // Optional, fügt der `render`-Funktion zusätzlich zu `form` Props hinzu
  props: {
    title: 'Child Form',
  },
  render: ({ form, title }) => {
    return (
      <div>
        <p>{title}</p>
        <form.AppField
          name="firstName"
          children={(field) => <field.TextField label="First Name" />}
        />
        <form.AppForm>
          <form.SubscribeButton label="Submit" />
        </form.AppForm>
      </div>
    )
  },
})

// /src/features/people/page.ts
const Parent = () => {
  const form = useAppForm({
    ...formOpts,
  })

  return <ChildForm form={form} title={'Testing'} />
}
```

## API-Nutzungsrichtlinien

Hier ist eine Tabelle, die Ihnen bei der Entscheidung hilft, welche APIs Sie verwenden sollten:

![](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/react_form_composability.svg)
