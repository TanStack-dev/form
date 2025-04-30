---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T22:01:28.814Z'
id: basic-concepts
title: Основные концепции
---

# Основные концепции и терминология

На этой странице представлены основные концепции и терминология, используемые в библиотеке `@tanstack/angular-form`. Ознакомление с этими концепциями поможет вам лучше понять и эффективнее работать с библиотекой.

## Экземпляр формы (Form Instance)

Экземпляр формы (Form Instance) — это объект, представляющий отдельную форму и предоставляющий методы и свойства для работы с ней. Экземпляр формы создаётся с помощью функции `injectForm`. Функция принимает объект с функцией `onSubmit`, которая вызывается при отправке формы.

```typescript
const form = injectForm({
  onSubmit: async ({ value }) => {
    // Do something with form data
    console.log(value)
  },
})
```

## Поле (Field)

Поле (Field) представляет отдельный элемент ввода формы, например, текстовое поле или чекбокс. Поля создаются с помощью директивы `tanstackField`. Директива принимает свойство `name`, которое должно соответствовать ключу в значениях формы по умолчанию. Она также предоставляет экземпляр `field` с внутренними свойствами, доступными через шаблонную переменную.

Пример:

```html
<ng-container [tanstackField]="form" name="firstName" #firstName="field">
  <input
    [value]="firstName.api.state.value"
    (blur)="firstName.api.handleBlur()"
    (input)="firstName.api.handleChange($any($event).target.value)"
  />
</ng-container>
```

## Состояние поля (Field State)

Каждое поле имеет собственное состояние (Field State), включающее текущее значение, статус валидации, сообщения об ошибках и другие метаданные. Состояние поля доступно через свойство `fieldApi.state`.

Пример:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Существует три состояния поля, которые помогают отслеживать взаимодействие пользователя с полем:

- Поле считается _"touched"_, когда пользователь кликает или переходит в него.
- Поле остается _"pristine"_, пока пользователь не изменит его значение.
- Поле становится _"dirty"_, после изменения значения.

Эти состояния можно проверить через флаги `isTouched`, `isPristine` и `isDirty`, как показано ниже.

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Состояния поля](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## API поля (Field API)

API поля (Field API) — это объект, доступный через свойство `tanstackField.api` при создании поля. Он предоставляет методы для работы с состоянием поля.

Пример:

```angular-html
<input
  [value]="fieldName.api.state.value"
  (blur)="fieldName.api.handleBlur()"
  (input)="fieldName.api.handleChange($any($event).target.value)"
/>
```

## Валидация (Validation)

`@tanstack/angular-form` поддерживает синхронную и асинхронную валидацию. Функции валидации передаются в директиву `tanstackField` через свойство `validators`.

Пример:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container [tanstackField]="form" name="firstName" #firstName="field">
      <input
        [value]="firstName.api.state.value"
        (blur)="firstName.api.handleBlur()"
        (input)="firstName.api.handleChange($any($event).target.value)"
      />
    </ng-container>
  `,
})
export class AppComponent {
  firstNameValidator: FieldValidateFn<any, any, string, any> = ({
                                                                       value,
                                                                     }) =>
    !value
      ? 'A first name is required'
      : value.length < 3
        ? 'First name must be at least 3 characters'
        : undefined

  firstNameAsyncValidator: FieldValidateAsyncFn<any, string, any> =
    async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && 'No "error" allowed in first name'
    }

  form = injectForm({
    defaultValues: {
      firstName: '',
    },
    onSubmit({ value }) {
      console.log(value)
    },
  })
}
```

## Валидация с использованием стандартных библиотек схем (Validation with Standard Schema Libraries)

Помимо ручной валидации, библиотека поддерживает спецификацию [Standard Schema](https://github.com/standard-schema/standard-schema).

Вы можете определить схему с помощью любой из библиотек, реализующих эту спецификацию, и передать её валидатору формы или поля.

Поддерживаемые библиотеки:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

Пример:

```angular-ts
import { z } from 'zod'

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="firstName"
      [validators]="{
        onChange: z.string().min(3, 'First name must be at least 3 characters'),
        onChangeAsyncDebounceMs: 500,
        onChangeAsync: firstNameAsyncValidator
      }"
      #firstName="field"
    >
      <!-- ... -->
    </ng-container>
  `,
})
export class AppComponent {
  firstNameAsyncValidator = z.string().refine(
    async (value) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return !value.includes('error')
    },
    {
      message: "No 'error' allowed in first name",
    },
  )

  form = injectForm({
    defaultValues: {
      firstName: '',
    },
    onSubmit({ value }) {
      // Do something with form data
      console.log(value)
    },
  })

  z = z
}
```

## Реактивность (Reactivity)

`@tanstack/angular-form` позволяет подписываться на изменения состояния формы и полей через `injectStore(this.form, selector)`.

Пример:

```typescript
import { injectForm, injectStore } from '@tanstack/angular-form'

@Component(/*...*/)
class AppComponent {
  form = injectForm(/*...*/)
  canSubmit = injectStore(this.form, (state) => state.canSubmit)
  isSubmitting = injectStore(this.form, (state) => state.isSubmitting)
}
```

## Обработчики событий (Listeners)

`@tanstack/angular-form` позволяет реагировать на определённые триггеры и "слушать" их для выполнения побочных эффектов.

Пример:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="country"
      [listeners]="{
        onChange: onCountryChange
      }"
      #country="field"
    ></ng-container>
  `,
})

...

onCountryChange: FieldListenerFn<any, any, any, any, string> = ({
    value,
  }) => {
    console.log(`Country changed to: ${value}, resetting province`)
    this.form.setFieldValue('province', '')
  }
```

Подробнее см. в [Listeners](./listeners.md)

## Поля-массивы (Array Fields)

Поля-массивы (Array Fields) позволяют управлять списками значений в форме, например, списком хобби. Поле-массив создаётся с помощью директивы `tanstackField`.

Для работы с полями-массивами можно использовать методы `pushValue`, `removeValue` и `swapValues` для добавления, удаления и перестановки значений.

Пример:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container [tanstackField]="form" name="hobbies" #hobbies="field">
      <div>
        Hobbies
        <div>
          @if (!hobbies.api.state.value.length) {
            No hobbies found
          }
          @for (_ of hobbies.api.state.value; track $index) {
            <div>
              <ng-container
                [tanstackField]="form"
                [name]="getHobbyName($index)"
                #hobbyName="field"
              >
                <div>
                  <label [for]="hobbyName.api.name">Name:</label>
                  <input
                    [id]="hobbyName.api.name"
                    [name]="hobbyName.api.name"
                    [value]="hobbyName.api.state.value"
                    (blur)="hobbyName.api.handleBlur()"
                    (input)="
                      hobbyName.api.handleChange($any($event).target.value)
                    "
                  />
                  <button
                    type="button"
                    (click)="hobbies.api.removeValue($index)"
                  >
                    X
                  </button>
                </div>
              </ng-container>
              <ng-container
                [tanstackField]="form"
                [name]="getHobbyDesc($index)"
                #hobbyDesc="field"
              >
                <div>
                  <label [for]="hobbyDesc.api.name">Description:</label>
                  <input
                    [id]="hobbyDesc.api.name"
                    [name]="hobbyDesc.api.name"
                    [value]="hobbyDesc.api.state.value"
                    (blur)="hobbyDesc.api.handleBlur()"
                    (input)="
                      hobbyDesc.api.handleChange($any($event).target.value)
                    "
                  />
                </div>
              </ng-container>
            </div>
          }
        </div>
        <button type="button" (click)="hobbies.api.pushValue(defaultHobby)">
          Add hobby
        </button>
      </div>
    </ng-container>
  `,
})
export class AppComponent {
  defaultHobby = {
    name: '',
    description: '',
    yearsOfExperience: 0,
  }

  getHobbyName = (idx: number) => `hobbies[${idx}].name` as const;
  getHobbyDesc = (idx: number) => `hobbies[${idx}].description` as const;

  form = injectForm({
    defaultValues: {
      hobbies: [] as Array<{
        name: string
        description: string
        yearsOfExperience: number
      }>,
    },
    onSubmit({ value }) {
      alert(JSON.stringify(value))
    },
  })
}
```

Это основные концепции и терминология, используемые в библиотеке `@tanstack/angular-form`. Понимание этих концепций поможет вам эффективнее работать с библиотекой и создавать сложные формы с лёгкостью.
