---
source-updated-at: '2025-03-01T08:30:20.000Z'
translation-updated-at: '2025-04-30T21:58:48.520Z'
id: quick-start
title: Быстрый старт
---

Минимальный набор для начала работы с TanStack Form — это создание контроллера `TanstackFormController`, как показано ниже, с интерфейсом `Employee` для нашей тестовой формы:

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

В этом примере `this` ссылается на экземпляр вашего `LitElement`, в котором вы хотите использовать TanStack Form.

Чтобы связать элемент формы в вашем шаблоне с TanStack Form, используйте метод `field` контроллера `TanstackFormController`.

Первый параметр `field` — это `FieldOptions`, а второй — колбэк для рендеринга вашего элемента.

```ts
field(FieldOptions, callback)
```

Готовая тестовая форма должна выглядеть примерно так, как показано ниже. Форма собирает имя пользователя из поля ввода:

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
    return html` <p>Пожалуйста, введите ваше имя></p>
      ${this.#form.field(
        {
          name: `firstName`,
          validators: {
            onChange: ({ value }) =>
              value.length < 3 ? 'Слишком короткое' : undefined,
          },
        },
        (field: FieldApi<Employee, 'firstName'>) => {
          return html` <div>
            <label class="first-name-label">Имя</label>
            <input
              id="firstName"
              type="text"
              placeholder="Имя"
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

Обратите внимание, что вам необходимо самостоятельно обрабатывать обновление элемента и формы, как показано в примере выше.
