---
source-updated-at: '2025-04-27T15:54:55.000Z'
translation-updated-at: '2025-04-30T21:53:47.721Z'
id: listeners
title: Listener
---

Für Situationen, in denen Sie auf Trigger "einwirken" oder "reagieren" möchten, gibt es die Listener-API. Beispielsweise, wenn Sie als Entwickler ein Formularfeld zurücksetzen möchten, nachdem sich ein anderes Feld geändert hat, würden Sie die Listener-API verwenden.

Stellen Sie sich folgenden Benutzerablauf vor:

- Benutzer wählt ein Land aus einer Dropdown-Liste aus.
- Benutzer wählt dann eine Provinz aus einer anderen Dropdown-Liste aus.
- Benutzer ändert das ausgewählte Land zu einem anderen.

In diesem Beispiel muss die ausgewählte Provinz zurückgesetzt werden, wenn der Benutzer das Land ändert, da sie nicht mehr gültig ist. Mit der Listener-API können wir das `onChange`-Event abonnieren und ein Reset für das Feld "province" auslösen, wenn der Listener aktiviert wird.

Events, auf die "gelauscht" werden kann, sind:

- `onChange`
- `onBlur`
- `onMount`
- `onSubmit`

```tsx
function App() {
  const form = useForm({
    defaultValues: {
      country: '',
      province: '',
    },
    // ...
  })

  return (
    <div>
      <form.Field
        name="country"
        listeners={{
          onChange: ({ value }) => {
            console.log(`Country changed to: ${value}, resetting province`)
            form.setFieldValue('province', '')
          },
        }}
      >
        {(field) => (
          <label>
            <div>Country</div>
            <input
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
          </label>
        )}
      </form.Field>

      <form.Field name="province">
        {(field) => (
          <label>
            <div>Province</div>
            <input
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
          </label>
        )}
      </form.Field>
    </div>
  )
}
```

### Integriertes Debouncing

Wenn Sie innerhalb eines Listeners eine API-Anfrage machen, möchten Sie möglicherweise die Aufrufe debouncen, da dies zu Leistungsproblemen führen kann.
Wir bieten eine einfache Methode zum Debouncen Ihrer Listener, indem Sie `onChangeDebounceMs` oder `onBlurDebounceMs` hinzufügen.

```tsx
<form.Field
  name="country"
  listeners={{
    onChangeDebounceMs: 500, // 500ms Debounce
    onChange: ({ value }) => {
      console.log(`Country changed to: ${value} without a change within 500ms, resetting province`)
      form.setFieldValue('province', '')
    },
  }}
>
  {(field) => (
    /* ... */
  )}
</form.Field>
```

### Formular-Listener

Auf einer höheren Ebene sind Listener auch auf Formularebene verfügbar, wodurch Sie Zugriff auf die Events `onMount` und `onSubmit` erhalten und `onChange` sowie `onBlur` an alle Kinder des Formulars weitergeleitet werden. Formular-Listener können ebenfalls wie zuvor beschrieben debounced werden.

`onMount`- und `onSubmit`-Listener haben folgende Props:

- `formApi`

`onChange`- und `onBlur`-Listener haben Zugriff auf:

- `fieldApi`
- `formApi`

```tsx
const form = useForm({
  listeners: {
    onMount: ({ formApi }) => {
      // Custom-Logging-Service
      loggingService('mount', formApi.state.values)
    },

    onChange: ({ formApi, fieldApi }) => {
      // Autosave-Logik
      if (formApi.state.isValid) {
        formApi.handleSubmit()
      }

      // fieldApi repräsentiert das Feld, das das Event ausgelöst hat.
      console.log(fieldApi.name, fieldApi.state.value)
    },
    onChangeDebounceMs: 500,
  },
})
```
