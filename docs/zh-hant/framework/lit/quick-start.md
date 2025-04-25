---
source-updated-at: '2025-03-01T08:30:20.000Z'
translation-updated-at: '2025-04-25T20:31:47.922Z'
id: quick-start
title: 快速開始
---
## 快速開始

使用 TanStack Form 的最低要求是建立一個 `TanstackFormController`，如下所示，並使用 `Employee` 介面作為測試表單的結構：

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

在此範例中，`this` 參照了您想要使用 TanStack Form 的 `LitElement` 實例。

要將模板中的表單元素與 TanStack Form 連接，請使用 `TanstackFormController` 的 `field` 方法。

`field` 的第一個參數是 `FieldOptions`，第二個參數是用於渲染元素的回呼函式。

```ts
field(FieldOptions, callback)
```

完成的測試表單應如下所示。該表單會從使用者輸入欄位收集名字：

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
    return html` <p>請輸入您的名字></p>
      ${this.#form.field(
        {
          name: `firstName`,
          validators: {
            onChange: ({ value }) =>
              value.length < 3 ? '長度不足' : undefined,
          },
        },
        (field: FieldApi<Employee, 'firstName'>) => {
          return html` <div>
            <label class="first-name-label">名字</label>
            <input
              id="firstName"
              type="text"
              placeholder="名字"
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

請注意，您需要自行處理元素和表單的更新，如上例所示。
