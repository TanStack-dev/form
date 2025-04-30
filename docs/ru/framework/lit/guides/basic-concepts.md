---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T22:01:45.279Z'
id: basic-concepts
title: Основные концепции
---

# Основные понятия и терминология

На этой странице представлены основные понятия и терминология, используемые в библиотеке `@tanstack/lit-form`. Ознакомление с этими концепциями поможет вам лучше понять и работать с библиотекой и её использованием в связке с Lit.

## Настройки формы (Form Options)

Вы можете создать настройки для вашей формы, чтобы их можно было использовать в нескольких формах, с помощью функции `formOptions`.

Например:

```tsx
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    employed: false,
    jobTitle: '',
  } as Employee,
})
```

## Экземпляр формы (Form Instance)

Экземпляр формы — это объект, который представляет отдельную форму и предоставляет методы и свойства для работы с ней. Вы создаёте экземпляр формы с помощью интерфейса `TanStackFormController`, предоставляемого `@tanstack/lit-form`. `TanStackFormController` инициализируется с текущим классом формы (`this`) и некоторыми настройками формы по умолчанию. Он инициализирует состояние формы, обрабатывает отправку формы и предоставляет методы для управления полями формы и их валидацией.

```tsx
#form = new TanStackFormController(this, {
  defaultValues: {
    firstName: '',
    lastName: '',
    employed: false,
    jobTitle: '',
  } as Employee,
})
```

Вы также можете создать экземпляр формы без использования `formOptions`, воспользовавшись автономным API `TanStackFormController`:

```tsx
#form = new TanStackFormController(this, {
  ...formOpts,
})
```

## Поле (Field)

Поле представляет собой отдельный элемент ввода формы, такой как текстовое поле или флажок. Поля создаются с помощью метода `field(FieldOptions, callback)`, предоставляемого экземпляром формы. Компонент принимает объект `FieldOptions` и функцию обратного вызова, которая получает объект `FieldApi`. Этот объект предоставляет методы для получения текущего значения поля, обработки изменений ввода и событий потери фокуса.

Например:

```ts
 ${this.#form.field(
    {
      name: `firstName`,
      validators: {
        onChange: ({ value }) =>
          value.length < 3 ? "Not long enough" : undefined,
        },
      },
      (field: FieldApi<Employee, "firstName">) => {
        return html` <div>
          <label class="first-name-label">First Name</label>
          <input
           id="firstName"
           type="text"
           class="first-name-input"
           placeholder="First Name"
           @blur="${() => field.handleBlur()}"
           .value="${field.state.value}"
           @input="${(event: InputEvent) => {
           if (event.currentTarget) {
            const newValue = (event.currentTarget as HTMLInputElement).value;
            field.handleChange(newValue);
           }
          }}"
        />
      </div>`;
    },
)}
```

## Состояние поля (Field State)

Каждое поле имеет собственное состояние, которое включает его текущее значение, статус валидации, сообщения об ошибках и другие метаданные. Вы можете получить доступ к состоянию поля через свойство `field.state`.

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Существуют три состояния поля, которые могут быть полезны для отслеживания взаимодействия пользователя с полем. Поле считается _"touched"_ (тронутым), когда пользователь кликает или переходит в него, _"pristine"_ (нетронутым) до изменения значения и _"dirty"_ (изменённым) после изменения значения. Вы можете проверить эти состояния через флаги `isTouched`, `isPristine` и `isDirty`, как показано ниже.

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Состояния поля](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)
