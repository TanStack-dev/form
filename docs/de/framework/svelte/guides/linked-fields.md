---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:56:23.725Z'
id: linked-fields
title: Verbundene Felder
---

## Zwei Formularfelder miteinander verknüpfen

Es kann vorkommen, dass Sie zwei Felder miteinander verknüpfen müssen, sodass eines validiert wird, sobald sich der Wert des anderen Feldes ändert.  
Ein typischer Anwendungsfall ist, wenn Sie sowohl ein `password`- als auch ein `confirm_password`-Feld haben,  
bei dem `confirm_password` einen Fehler anzeigen soll, wenn der Wert nicht mit `password` übereinstimmt –  
unabhängig davon, welches Feld die Änderung ausgelöst hat.

Stellen Sie sich folgenden Benutzerfluss vor:

- Der Benutzer aktualisiert das Bestätigungs-Passwort-Feld.
- Der Benutzer aktualisiert das Haupt-Passwort-Feld.

In diesem Beispiel würden weiterhin Fehler im Formular vorhanden sein,  
da die Validierung des "Bestätigungs-Passwort"-Feldes nicht erneut ausgeführt wurde, um es als akzeptiert zu markieren.

Um dies zu lösen, müssen wir sicherstellen, dass die "Bestätigungs-Passwort"-Validierung erneut ausgeführt wird, wenn das Passwort-Feld aktualisiert wird.  
Dazu können Sie dem `confirm_password`-Feld eine `onChangeListenTo`-Eigenschaft hinzufügen.

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      password: '',
      confirm_password: '',
    },
    // ...
  }))
</script>

<div>
  <form.Field name="password">
    {#snippet children(field)}
      <label>
        <div>Password</div>
        <input
          value={field.state.value}
          onChange={(e) => field.handleChange(e.target.value)}
        />
      </label>
    {/snippet}
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
    {#snippet children(field)}
      <div>
        <label>
          <div>Confirm Password</div>
          <input
            value={field.state.value}
            onchange={(e) => field.handleChange(e.target.value)}
          />
        </label>
        {#each field.state.meta.errors as err}
          <div>{err}</div>
        {/each}
      </div>
    {/snippet}
  </form.Field>
</div>
```

Dies funktioniert ähnlich mit der `onBlurListenTo`-Eigenschaft, die die Validierung erneut ausführt, wenn das Feld den Fokus verliert.
