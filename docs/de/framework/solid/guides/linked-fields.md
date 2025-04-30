---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:55:43.522Z'
id: linked-fields
title: Verbundene Felder
---

## Zwei Formularfelder miteinander verknüpfen

Möglicherweise müssen Sie zwei Felder miteinander verknüpfen, wobei eines validiert wird, sobald sich der Wert eines anderen Feldes ändert.  
Ein typischer Anwendungsfall ist, wenn Sie sowohl ein `password`- als auch ein `confirm_password`-Feld haben.  
Hier soll `confirm_password` einen Fehler anzeigen, wenn der Wert nicht mit dem `password`-Feld übereinstimmt – unabhängig davon, welches Feld die Änderung ausgelöst hat.

Stellen Sie sich folgenden Benutzerablauf vor:

- Der Benutzer aktualisiert das Bestätigungs-Passwort-Feld.
- Der Benutzer aktualisiert das eigentliche Passwort-Feld.

In diesem Beispiel bleiben Fehler im Formular bestehen,  
da die Validierung des "Bestätigungs-Passwort"-Felds nicht erneut ausgeführt wurde, um es als akzeptiert zu markieren.

Um dies zu lösen, müssen wir sicherstellen, dass die "Bestätigungs-Passwort"-Validierung erneut ausgeführt wird, wenn das Passwort-Feld aktualisiert wird.  
Dazu können Sie dem `confirm_password`-Feld eine `onChangeListenTo`-Eigenschaft hinzufügen.

```tsx
export default function App() {
  const form = createForm(() => ({
    defaultValues: {
      password: '',
      confirm_password: '',
    },
    // ...
  }))

  return (
    <div>
      <form.Field name="password">
        {(field) => (
          <label>
            <div>Password</div>
            <input
              value={field().state.value}
              onChange={(e) => field().handleChange(e.target.value)}
            />
          </label>
        )}
      </form.Field>
      <form.Field
        name="confirm_password"
        validators={{
          onChangeListenTo: ['password'],
          onChange: ({ value, fieldApi }) => {
            if (value !== fieldApi.form.getFieldValue('password')) {
              return 'Passwords do not match'
            }
            return undefined
          },
        }}
      >
        {(field) => (
          <div>
            <label>
              <div>Confirm Password</div>
              <input
                value={field().state.value}
                onChange={(e) => field().handleChange(e.target.value)}
              />
            </label>
            <Index each={field().state.meta.errors}>
              {(err) => <div>{err()}</div>}
            </Index>
          </div>
        )}
      </form.Field>
    </div>
  )
}
```

Dies funktioniert ähnlich mit der `onBlurListenTo`-Eigenschaft, die die Validierung erneut ausführt, wenn das Feld den Fokus verliert.
