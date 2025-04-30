---
source-updated-at: '2025-03-01T08:30:20.000Z'
translation-updated-at: '2025-04-30T21:24:26.962Z'
id: quick-start
title: クイックスタート
---

# クイックスタート

TanStack Formを始めるための最低限の要件は、テストフォーム用の`Employee`インターフェースと共に以下のように`TanstackFormController`を作成することです：

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

この例では、`this`はTanStack Formを使用したい`LitElement`のインスタンスを参照しています。

テンプレート内のフォーム要素をTanStack Formと接続するには、`TanstackFormController`の`field`メソッドを使用します。

`field`の最初のパラメータは`FieldOptions`で、2番目は要素をレンダリングするコールバックです。

```ts
field(FieldOptions, callback)
```

完成したテストフォームは以下のようになります。このフォームはユーザーの入力フィールドから名（first name）を収集します：

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
    return html` <p>Please enter your first name></p>
      ${this.#form.field(
        {
          name: `firstName`,
          validators: {
            onChange: ({ value }) =>
              value.length < 3 ? 'Not long enough' : undefined,
          },
        },
        (field: FieldApi<Employee, 'firstName'>) => {
          return html` <div>
            <label class="first-name-label">First Name</label>
            <input
              id="firstName"
              type="text"
              placeholder="First Name"
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

上記の例で示されているように、要素とフォームの更新を自分で処理する必要があることに注意してください。
