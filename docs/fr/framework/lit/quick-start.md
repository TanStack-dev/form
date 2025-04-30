---
source-updated-at: '2025-03-01T08:30:20.000Z'
translation-updated-at: '2025-04-30T21:34:46.367Z'
id: quick-start
title: Démarrage rapide
---

# Démarrage rapide

Le strict minimum pour commencer avec TanStack Form est de créer un `TanstackFormController` comme illustré ci-dessous avec l'interface `Employee` pour notre formulaire de test :

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

Dans cet exemple, `this` fait référence à l'instance de votre `LitElement` dans laquelle vous souhaitez utiliser TanStack Form.

Pour connecter un élément de formulaire dans votre template avec TanStack Form, utilisez la méthode `field` de `TanstackFormController`.

Le premier paramètre de `field` est `FieldOptions` et le second est une fonction de rappel (callback) pour afficher votre élément.

```ts
field(FieldOptions, callback)
```

Notre formulaire de test final devrait ressembler à ce qui suit. Le formulaire collecte le prénom depuis un champ de saisie utilisateur :

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
    return html` <p>Veuillez saisir votre prénom></p>
      ${this.#form.field(
        {
          name: `firstName`,
          validators: {
            onChange: ({ value }) =>
              value.length < 3 ? 'Pas assez long' : undefined,
          },
        },
        (field: FieldApi<Employee, 'firstName'>) => {
          return html` <div>
            <label class="first-name-label">Prénom</label>
            <input
              id="firstName"
              type="text"
              placeholder="Prénom"
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

Notez que vous devez gérer vous-même la mise à jour de l'élément et du formulaire comme illustré dans l'exemple ci-dessus.
