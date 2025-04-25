---
source-updated-at: '2025-03-01T08:30:20.000Z'
translation-updated-at: '2025-04-12T04:08:31.027Z'
id: quick-start
title: 快速开始
---
## 快速入门

开始使用 TanStack Form 的最低要求是创建一个 `TanstackFormController`，如下所示，我们使用 `Employee` 接口作为测试表单的数据结构：

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

在此示例中，`this` 引用了您想要使用 TanStack Form 的 `LitElement` 实例。

要将模板中的表单元素与 TanStack Form 关联，请使用 `TanstackFormController` 的 `field` 方法。

`field` 方法的第一个参数是 `FieldOptions`，第二个参数是用于渲染元素的回调函数。

```ts
field(FieldOptions, callback)
```

我们完成的测试表单应如下所示。该表单通过用户输入字段收集名字：

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
    return html` <p>请输入您的名字></p>
      ${this.#form.field(
        {
          name: `firstName`,
          validators: {
            onChange: ({ value }) =>
              value.length < 3 ? '长度不足' : undefined,
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

请注意，您需要像上面的示例一样自行处理元素和表单的更新。
