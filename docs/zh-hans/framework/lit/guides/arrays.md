---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-12T04:11:36.599Z'
id: arrays
title: 数组
---

TanStack Form 支持将数组作为表单值使用，包括数组内的子对象值。

## 基本用法

要使用数组，您可以在数组值上使用 `field.state.value`，例如：

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
        <h1>请填写您的信息</h1>
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

当您在字段上运行 pushValue 时，这将生成映射后的 HTML：

```html
<div class="container">
  <button type="button" @click="${()" ="">
    { peopleField.pushValue({name: "",age: ""}) }}> 添加人员
  </button>
</div>
```

最后，您可以像这样使用子字段：

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

## 完整示例

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
        <h1>请填写您的信息</h1>
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
                  添加人员
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
            重置
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
