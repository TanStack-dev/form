---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-25T20:47:51.581Z'
id: arrays
title: 陣列
---

TanStack Form 支援將陣列 (array) 作為表單值使用，包含陣列中的子物件值。

## 基本用法

要使用陣列，你可以對陣列值使用 `field.state.value`，如下所示：

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
        <h1>請輸入您的詳細資料</h1>
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

這會在每次對欄位執行 pushValue 時生成對應的 HTML：

```html
<div class="container">
  <button type="button" @click="${()" ="">
    { peopleField.pushValue({name: "",age: ""}) }}> 新增人員
  </button>
</div>
```

最後，你可以像這樣使用子欄位 (subfield)：

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
          placeholder="姓名"
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

## 完整範例

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
        <h1>請輸入您的詳細資料</h1>
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
                            <label>姓名</label>
                            <input
                              type="text"
                              placeholder="姓名"
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
                  新增人員
                </button>
              </div> `;
          }
        )}

        <div class="container">
          <button type="submit" ?disabled=${this.#form.api.state.isSubmitting}>
            ${this.#form.api.state.isSubmitting ? html` 提交中` : "提交"}
          </button>
          <button
            type="button"
            id="reset"
            @click=${() => {
              this.#form.api.reset();
            }}
          >
            重設
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
