---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:54:28.173Z'
id: linked-fields
title: Verbundene Felder
---

# Zwei Formularfelder miteinander verknüpfen

Es kann vorkommen, dass Sie zwei Felder miteinander verknüpfen müssen, sodass eines validiert wird, sobald sich der Wert eines anderen Feldes ändert.  
Ein typischer Anwendungsfall ist, wenn Sie sowohl ein `password`- als auch ein `confirm_password`-Feld haben,  
wobei `confirm_password` einen Fehler anzeigen soll, wenn der Wert nicht mit dem `password`-Feld übereinstimmt –  
unabhängig davon, welches Feld die Änderung ausgelöst hat.

Stellen Sie sich folgenden Benutzerablauf vor:

- Der Benutzer aktualisiert das Bestätigungs-Passwort-Feld.
- Der Benutzer aktualisiert das Haupt-Passwort-Feld.

In diesem Beispiel würden weiterhin Fehler im Formular vorhanden sein,  
da die Validierung des "Bestätigungs-Passwort"-Feldes nicht erneut ausgeführt wurde, um es als korrekt zu markieren.

Um dies zu lösen, müssen wir sicherstellen, dass die Validierung des "Bestätigungs-Passwort"-Feldes erneut ausgeführt wird, wenn das Passwort-Feld aktualisiert wird.  
Dazu können Sie dem `confirm_password`-Feld eine `onChangeListenTo`-Eigenschaft hinzufügen.

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    password: '',
    confirm_password: '',
  },
  // ...
})
</script>

<template>
  <div>
    <form @submit.prevent.stop="form.handleSubmit">
      <div>
        <form.Field name="password">
          <template v-slot="{ field }">
            <div>Password:</div>
            <input
              :value="field.state.value"
              @input="
                (e) => field.handleChange((e.target as HTMLInputElement).value)
              "
            />
          </template>
        </form.Field>
        <form.Field
          name="confirm_password"
          :validators="{
            onChangeListenTo: ['password'],
            onChange: ({ value, fieldApi }) => {
              if (value !== fieldApi.form.getFieldValue('password')) {
                return 'Passwords do not match'
              }
              return undefined
            },
          }"
        >
          <template v-slot="{ field }">
            <div>Confirm Password:</div>
            <input
              :value="field.state.value"
              @input="
                (e) => field.handleChange((e.target as HTMLInputElement).value)
              "
            />
            <div v-for="(err, index) in field.state.meta.errors" :key="index">
              {{ err }}
            </div>
          </template>
        </form.Field>
      </div>
      <button type="submit">Submit</button>
    </form>
  </div>
</template>
```

Dies funktioniert ähnlich mit der `onBlurListenTo`-Eigenschaft, die die Validierung erneut ausführt, wenn das Feld den Fokus verliert.
