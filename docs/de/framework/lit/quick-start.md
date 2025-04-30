---
source-updated-at: '2025-03-01T08:30:20.000Z'
translation-updated-at: '2025-04-30T21:52:51.625Z'
id: quick-start
title: Schnellstart
---

# Schnellstart

Das absolute Minimum, um mit TanStack Form zu beginnen, ist die Erstellung eines `TanstackFormController`, wie unten mit der `Employee`-Schnittstelle für unser Testformular zu sehen:

```ts
interface Employee {
  firstName: string
  lastName: string
  employed: boolean
  jobTitle: string
}

#form = new TanStackFormController()<Employee>(this, {
  defaultValues: {
    firstName: '',
    lastName: '',
    employed: false,
    jobTitle: '',
  },
})
```

In diesem Beispiel verweist `this` auf die Instanz Ihres `LitElement`, in dem Sie TanStack Form verwenden möchten.

Um ein Formularelement in Ihrer Vorlage mit TanStack Form zu verbinden, verwenden Sie die `field`-Methode des `TanstackFormController`.

Der erste Parameter von `field` ist `FieldOptions` und der zweite ist ein Callback zum Rendern Ihres Elements.

```ts
field(FieldOptions, callback)
```

Unser fertiges Testformular sollte etwa wie folgt aussehen. Das Formular sammelt den Vornamen aus einem Benutzereingabefeld:

```ts
export class TestForm extends LitElement {
  #form = new TanStackFormController<Employee>(this, {
    defaultValues: {
      firstName: '',
      lastName: '',
      employed: false,
      jobTitle: '',
    },
  })
  render() {
    return html` <p>Bitte geben Sie Ihren Vornamen ein></p>
      ${this.#form.field(
        {
          name: `firstName`,
          validators: {
            onChange: ({ value }) =>
              value.length < 3 ? 'Nicht lang genug' : undefined,
          },
        },
        (field: FieldApi<Employee, 'firstName'>) => {
          return html` <div>
            <label class="first-name-label">Vorname</label>
            <input
              id="firstName"
              type="text"
              placeholder="Vorname"
              @blur="${() => field.handleBlur()}"
              .value="${field.getValue()}"
              @input="${(event: InputEvent) => {
                if (event.currentTarget) {
                  const newValue = (event.currentTarget as HTMLInputElement)
                    .value
                  field.handleChange(newValue)
                }
              }}"
            />
          </div>`
        },
      )}`
  }
}
```

Beachten Sie, dass Sie die Aktualisierung des Elements und des Formulars selbst übernehmen müssen, wie im obigen Beispiel gezeigt.
