---
source-updated-at: '2025-03-04T14:23:57.000Z'
translation-updated-at: '2025-04-30T21:54:24.756Z'
id: custom-errors
title: Benutzerdefinierte Fehler
---

# Benutzerdefinierte Fehler (Custom Errors)

TanStack Form bietet vollständige Flexibilität bei den Arten von Fehlerwerten, die Sie von Validierungen zurückgeben können. String-Fehler sind am gebräuchlichsten und einfach zu handhaben, aber die Bibliothek erlaubt es Ihnen, jeden beliebigen Werttyp von Ihren Validierungen zurückzugeben.

Als allgemeine Regel gilt: Jeder truthy-Wert wird als Fehler betrachtet und markiert das Formular oder Feld als ungültig, während falsy-Werte (`false`, `undefined`, `null`, etc.) bedeuten, dass kein Fehler vorliegt und das Formular oder Feld gültig ist.

## String-Werte aus Formularen zurückgeben

```tsx
<form.Field
  name="username"
  validators={{
    onChange: ({ value }) =>
      value.length < 3 ? 'Username must be at least 3 characters' : undefined,
  }}
/>
```

Für formularweite Validierungen, die mehrere Felder betreffen:

```tsx
const form = useForm({
  defaultValues: {
    username: '',
    email: '',
  },
  validators: {
    onChange: ({ value }) => {
      return {
        fields: {
          username:
            value.username.length < 3 ? 'Username too short' : undefined,
          email: !value.email.includes('@') ? 'Invalid email' : undefined,
        },
      }
    },
  },
})
```

String-Fehler sind der häufigste Typ und lassen sich einfach in Ihrer Benutzeroberfläche anzeigen:

```tsx
{
  field.state.meta.errors.map((error, i) => (
    <div key={i} className="error">
      {error}
    </div>
  ))
}
```

### Zahlen

Nützlich für die Darstellung von Mengen, Schwellwerten oder Größenordnungen:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 18 ? 18 - value : undefined),
  }}
/>
```

Anzeige in der Benutzeroberfläche:

```tsx
{
  /* TypeScript weiß, dass der Fehler eine Zahl ist, basierend auf Ihrer Validierung */
}
;<div className="error">
  You need {field.state.meta.errors[0]} more years to be eligible
</div>
```

### Booleans

Einfache Flags, um den Fehlerzustand anzuzeigen:

```tsx
<form.Field
  name="accepted"
  validators={{
    onChange: ({ value }) => (!value ? true : undefined),
  }}
/>
```

Anzeige in der Benutzeroberfläche:

```tsx
{
  field.state.meta.errors[0] === true && (
    <div className="error">You must accept the terms</div>
  )
}
```

### Objekte

Umfangreiche Fehlerobjekte mit mehreren Eigenschaften:

```tsx
<form.Field
  name="email"
  validators={{
    onChange: ({ value }) => {
      if (!value.includes('@')) {
        return {
          message: 'Invalid email format',
          severity: 'error',
          code: 1001,
        }
      }
      return undefined
    },
  }}
/>
```

Anzeige in der Benutzeroberfläche:

```tsx
{
  typeof field.state.meta.errors[0] === 'object' && (
    <div className={`error ${field.state.meta.errors[0].severity}`}>
      {field.state.meta.errors[0].message}
      <small> (Code: {field.state.meta.errors[0].code})</small>
    </div>
  )
}
```

Im obigen Beispiel hängt es von dem Ereignisfehler ab, den Sie anzeigen möchten.

### Arrays

Mehrere Fehlermeldungen für ein einzelnes Feld:

```tsx
<form.Field
  name="password"
  validators={{
    onChange: ({ value }) => {
      const errors = []
      if (value.length < 8) errors.push('Password too short')
      if (!/[A-Z]/.test(value)) errors.push('Missing uppercase letter')
      if (!/[0-9]/.test(value)) errors.push('Missing number')

      return errors.length ? errors : undefined
    },
  }}
/>
```

Anzeige in der Benutzeroberfläche:

```tsx
{
  Array.isArray(field.state.meta.errors) && (
    <ul className="error-list">
      {field.state.meta.errors.map((err, i) => (
        <li key={i}>{err}</li>
      ))}
    </ul>
  )
}
```

## Die `disableErrorFlat`-Property für Felder

Standardmäßig fasst TanStack Form Fehler aus allen Validierungsquellen (onChange, onBlur, onSubmit) in einem einzigen `errors`-Array zusammen. Die `disableErrorFlat`-Property bewahrt die Fehlerquellen:

```tsx
<form.Field
  name="email"
  disableErrorFlat
  validators={{
    onChange: ({ value }) =>
      !value.includes('@') ? 'Invalid email format' : undefined,
    onBlur: ({ value }) =>
      !value.endsWith('.com') ? 'Only .com domains allowed' : undefined,
    onSubmit: ({ value }) => (value.length < 5 ? 'Email too short' : undefined),
  }}
/>
```

Ohne `disableErrorFlat` würden alle Fehler in `field.state.meta.errors` kombiniert. Mit dieser Property können Sie auf Fehler nach ihrer Quelle zugreifen:

```tsx
{
  field.state.meta.errorMap.onChange && (
    <div className="real-time-error">{field.state.meta.errorMap.onChange}</div>
  )
}

{
  field.state.meta.errorMap.onBlur && (
    <div className="blur-feedback">{field.state.meta.errorMap.onBlur}</div>
  )
}

{
  field.state.meta.errorMap.onSubmit && (
    <div className="submit-error">{field.state.meta.errorMap.onSubmit}</div>
  )
}
```

Dies ist nützlich für:

- Anzeige verschiedener Fehlertypen mit unterschiedlichen UI-Behandlungen
- Priorisierung von Fehlern (z.B. deutlichere Anzeige von Übermittlungsfehlern)
- Implementierung einer progressiven Offenlegung von Fehlern

## Typsicherheit von `errors` und `errorMap`

TanStack Form bietet starke Typsicherheit für die Fehlerbehandlung. Jeder Schlüssel in der `errorMap` hat genau den Typ, der von der entsprechenden Validierung zurückgegeben wird, während das `errors`-Array einen Union-Typ aller möglichen Fehlerwerte aus allen Validierungen enthält:

```tsx
<form.Field
  name="password"
  validators={{
    onChange: ({ value }) => {
      // Gibt einen String oder undefined zurück
      return value.length < 8 ? 'Too short' : undefined
    },
    onBlur: ({ value }) => {
      // Gibt ein Objekt oder undefined zurück
      if (!/[A-Z]/.test(value)) {
        return { message: 'Missing uppercase', level: 'warning' }
      }
      return undefined
    },
  }}
  children={(field) => {
    // TypeScript weiß, dass errors[0] string | { message: string, level: string } | undefined sein kann
    const error = field.state.meta.errors[0]

    // Typsichere Fehlerbehandlung
    if (typeof error === 'string') {
      return <div className="string-error">{error}</div>
    } else if (error && typeof error === 'object') {
      return <div className={error.level}>{error.message}</div>
    }

    return null
  }}
/>
```

Die `errorMap`-Property ist ebenfalls vollständig typisiert und entspricht den Rückgabetypen Ihrer Validierungsfunktionen:

```tsx
// Mit disableErrorFlat
<form.Field
  name="email"
  disableErrorFlat
  validators={{
    onChange: ({ value }): string | undefined =>
      !value.includes("@") ? "Invalid email" : undefined,
    onBlur: ({ value }): { code: number, message: string } | undefined =>
      !value.endsWith(".com") ? { code: 100, message: "Wrong domain" } : undefined
  }}
  children={(field) => {
    // TypeScript kennt den genauen Typ jeder Fehlerquelle
    const onChangeError: string | undefined = field.state.meta.errorMap.onChange;
    const onBlurError: { code: number, message: string } | undefined = field.state.meta.errorMap.onBlur;

    return (/* ... */);
  }}
/>
```

Diese Typsicherheit hilft, Fehler bereits zur Kompilierzeit statt zur Laufzeit zu erkennen, was Ihren Code zuverlässiger und wartbarer macht.
