---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:54:00.470Z'
id: ui-libraries
title: UI-Bibliotheken
---

## Verwendung von TanStack Form mit UI-Bibliotheken

TanStack Form ist eine headless Bibliothek (headless library), die Ihnen vollständige Flexibilität bei der Gestaltung bietet. Es ist mit einer Vielzahl von UI-Bibliotheken kompatibel, darunter `Tailwind`, `Material UI`, `Mantine` oder sogar einfaches CSS.

Dieser Leitfaden konzentriert sich auf `Material UI` und `Mantine`, aber die Konzepte sind auf jede UI-Bibliothek Ihrer Wahl übertragbar.

### Voraussetzungen

Bevor Sie TanStack Form mit einer UI-Bibliothek integrieren, stellen Sie sicher, dass die erforderlichen Abhängigkeiten in Ihrem Projekt installiert sind:

- Für `Material UI` folgen Sie den Installationsanweisungen auf der [offiziellen Website](https://mui.com/material-ui/getting-started/).
- Für `Mantine` lesen Sie die [Dokumentation](https://mantine.dev/).

Hinweis: Obwohl Sie Bibliotheken mischen können, ist es generell ratsam, bei einer zu bleiben, um Konsistenz zu gewährleisten und Überladung zu minimieren.

### Beispiel mit Mantine

Hier ist ein Beispiel, das die Integration von TanStack Form mit Mantine-Komponenten demonstriert:

```tsx
import { TextInput, Checkbox } from '@mantine/core'
import { useForm } from '@tanstack/react-form'

export default function App() {
  const { Field, handleSubmit, state } = useForm({
    defaultValues: {
      firstName: '',
      lastName: '',
      isChecked: false,
    },
    onSubmit: async ({ value }) => {
      // Handle form submission
      console.log(value)
    },
  })

  return (
    <>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          handleSubmit()
        }}
      >
        <Field
          name="firstName"
          children={({ state, handleChange, handleBlur }) => (
            <TextInput
              defaultValue={state.value}
              onChange={(e) => handleChange(e.target.value)}
              onBlur={handleBlur}
              placeholder="Enter your name"
            />
          )}
        />
        <Field
          name="isChecked"
          children={({ state, handleChange, handleBlur }) => (
            <Checkbox
              onChange={(e) => handleChange(e.target.checked)}
              onBlur={handleBlur}
              checked={state.value}
            />
          )}
        />
      </form>
      <div>
        <pre>{JSON.stringify(state.values, null, 2)}</pre>
      </div>
    </>
  )
}
```

- Zunächst verwenden wir den `useForm`-Hook von TanStack und destrukturieren die erforderlichen Eigenschaften. Dieser Schritt ist optional; alternativ könnten Sie `const form = useForm()` verwenden, wenn Sie es bevorzugen. Die Typinferenz von TypeScript gewährleistet eine reibungslose Erfahrung, unabhängig vom Ansatz.
- Die `Field`-Komponente, die von `useForm` abgeleitet wird, akzeptiert mehrere Eigenschaften, wie z.B. `validators`. Für diese Demonstration konzentrieren wir uns auf zwei primäre Eigenschaften: `name` und `children`.
  - Die `name`-Eigenschaft identifiziert jedes `Field`, zum Beispiel `firstName` in unserem Beispiel.
  - Die `children`-Eigenschaft nutzt das Konzept der Render-Props (render props), das uns erlaubt, Komponenten ohne unnötige Abstraktionen zu integrieren.
- Das Design von TanStack basiert stark auf Render-Props und bietet Zugriff auf `children` innerhalb der `Field`-Komponente. Dieser Ansatz ist vollständig typsicher. Bei der Integration mit Mantine-Komponenten, wie `TextInput`, destrukturieren wir selektiv Eigenschaften wie `state.value`, `handleChange` und `handleBlur`. Dieser selektive Ansatz ist aufgrund der leichten Unterschiede in den Typen zwischen `TextInput` und dem `field`, das wir in den children erhalten, notwendig.
- Indem Sie diese Schritte befolgen, können Sie Mantine-Komponenten nahtlos mit TanStack Form integrieren.
- Diese Methodik ist ebenso auf andere Komponenten wie `Checkbox` anwendbar und gewährleistet eine konsistente Integration über verschiedene UI-Elemente hinweg.

### Verwendung mit Material UI

Der Prozess zur Integration von Material UI-Komponenten ist ähnlich. Hier ist ein Beispiel mit TextField und Checkbox von Material UI:

```tsx
        <Field
            name="lastName"
            children={({ state, handleChange, handleBlur }) => {
              return (
                <TextField
                  id="filled-basic"
                  label="Filled"
                  variant="filled"
                  defaultValue={state.value}
                  onChange={(e) => handleChange(e.target.value)}
                  onBlur={handleBlur}
                  placeholder="Enter your last name"
                />
              );
            }}
          />

           <Field
            name="isMuiCheckBox"
            children={({ state, handleChange, handleBlur }) => {
              return (
                <MuiCheckbox
                  onChange={(e) => handleChange(e.target.checked)}
                  onBlur={handleBlur}
                  checked={state.value}
                />
              );
            }}
          />

```

- Der Integrationsansatz ist derselbe wie bei Mantine.
- Der Hauptunterschied liegt in den spezifischen Eigenschaften und Styling-Optionen der Material UI-Komponenten.
