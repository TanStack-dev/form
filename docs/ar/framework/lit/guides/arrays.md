---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T22:07:41.742Z'
id: arrays
title: المصفوفات
---

يدعم TanStack Form المصفوفات كقيم في النموذج، بما في ذلك القيم الفرعية داخل المصفوفة.

## الاستخدام الأساسي

لاستخدام مصفوفة، يمكنك استخدام `field.state.value` على قيمة المصفوفة، كما في:

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
        <h1>Please enter your details</h1>
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

سيؤدي هذا إلى إنشاء HTML المعين في كل مرة تقوم فيها بتشغيل pushValue على الحقل:

```html
<div class="container">
  <button type="button" @click="${()" ="">
    { peopleField.pushValue({name: "",age: ""}) }}> Add Person
  </button>
</div>
```

أخيرًا، يمكنك استخدام حقل فرعي كما يلي:

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
          placeholder="Name"
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

## مثال كامل

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
        <h1>Please enter your details</h1>
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
                            <label>Name</label>
                            <input
                              type="text"
                              placeholder="Name"
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
                  Add Person
                </button>
              </div> `;
          }
        )}

        <div class="container">
          <button type="submit" ?disabled=${this.#form.api.state.isSubmitting}>
            ${this.#form.api.state.isSubmitting ? html` Submitting` : "Submit"}
          </button>
          <button
            type="button"
            id="reset"
            @click=${() => {
              this.#form.api.reset();
            }}
          >
            Reset
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
