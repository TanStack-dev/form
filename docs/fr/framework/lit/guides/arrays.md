---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:37:55.702Z'
id: arrays
title: Tableaux
---

TanStack Form prend en charge les tableaux comme valeurs dans un formulaire, y compris les valeurs de sous-objets à l'intérieur d'un tableau.

## Utilisation de base

Pour utiliser un tableau, vous pouvez utiliser `field.state.value` sur une valeur de tableau, comme dans :

```ts
export class TestForm extends LitElement {
  #form = new TanStackFormController(this, {
    defaultValues: {
      people: [] as Array<{ name: string; age: string }>,
    },
  })
  render() {
    return html`
      <form
        id="form"
        @submit=${(e: Event) => {
          e.preventDefault()
        }}
      >
        <h1>Veuillez entrer vos informations</h1>
        ${this.#form.field(
          {
            name: `people`,
          },
          (peopleField) => {
            return html`${repeat(
              peopleField.state.value,
              (_, index) => index,
              (_, index) => {
                return html` // ... `
              },
            )} `
          },
        )}
      </form>
    `
  }
}
```

Cela générera le HTML mappé chaque fois que vous exécutez pushValue sur le champ :

```html
<div class="container">
  <button type="button" @click="${()" ="">
    { peopleField.pushValue({name: "",age: ""}) }}> Ajouter une personne
  </button>
</div>
```

Enfin, vous pouvez utiliser un sous-champ comme ceci :

```ts
return html`
  ${this.#form.field(
    {
      name: `people[${index}].name`,
    },
    (field) => {
      return html`
        <input
          type="text"
          placeholder="Nom"
          .value="${field.state.value}"
          @input="${(e: Event) => {
            const target = e.target as HTMLInputElement
            field.handleChange(target.value)
          }}"
        />
      `
    },
  )}
`
```

## Exemple complet

```typescript
export class TestForm extends LitElement {
  #form = new TanStackFormController(this, {
    defaultValues: {
      people: [] as Array<{ name: string}>,
    },
  });
  render() {
    return html`
      <form
        id="form"
        @submit=${(e: Event) => {
          e.preventDefault();
        }}
      >
        <h1>Veuillez entrer vos informations</h1>
        ${this.#form.field(
          {
            name: `people`,
          },
          (peopleField) => {
            return html`${repeat(
                peopleField.state.value,
                (_, index) => index,
                (_, index) => {
                  return html`
                    ${this.#form.field(
                      {
                        name: `people[${index}].name`,
                      },
                      (field) => {
                        return html` <div>
                          <div class="container">
                            <label>Nom</label>
                            <input
                              type="text"
                              placeholder="Nom"
                              .value="${field.state.value}"
                              @input="${(e: Event) => {
                                const target = e.target as HTMLInputElement;
                                field.handleChange(target.value);
                              }}"
                            />
                          </div>
                        </div>`;
                      }
                    )}
                  `;
                }
              )}

              <div class="container">
                <button
                  type="button"
                  @click=${() => {
                    peopleField.pushValue({
                      name: "",
                    });
                  }}
                >
                  Ajouter une personne
                </button>
              </div> `;
          }
        )}

        <div class="container">
          <button type="submit" ?disabled=${this.#form.api.state.isSubmitting}>
            ${this.#form.api.state.isSubmitting ? html` Envoi en cours` : "Envoyer"}
          </button>
          <button
            type="button"
            id="reset"
            @click=${() => {
              this.#form.api.reset();
            }}
          >
            Réinitialiser
          </button>
        </div>
      </form>
    `;
  }

declare global {
  interface HTMLElementTagNameMap {
    "test-form": TestForm;
  }
}
```
