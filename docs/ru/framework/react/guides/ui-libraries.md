---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:59:58.683Z'
id: ui-libraries
title: UI-библиотеки
---

## Использование TanStack Form с UI-библиотеками

TanStack Form — это headless-библиотека (headless library), предоставляющая полную свободу в стилизации. Она совместима с широким спектром UI-библиотек, включая `Tailwind`, `Material UI`, `Mantine` или даже обычный CSS.

В этом руководстве основное внимание уделяется `Material UI` и `Mantine`, но представленные концепции применимы к любой UI-библиотеке на ваш выбор.

### Предварительные требования

Перед интеграцией TanStack Form с UI-библиотекой убедитесь, что необходимые зависимости установлены в вашем проекте:

- Для `Material UI` следуйте инструкциям по установке на [официальном сайте](https://mui.com/material-ui/getting-started/).
- Для `Mantine` обратитесь к [документации](https://mantine.dev/).

Примечание: Хотя можно комбинировать библиотеки, рекомендуется придерживаться одной для обеспечения согласованности и минимизации избыточного кода.

### Пример с Mantine

Вот пример интеграции TanStack Form с компонентами Mantine:

```tsx
import { TextInput, Checkbox } from '@mantine/core'
import { useForm } from '@tanstack/react-form'

export default function App() {
  const { Field, handleSubmit, state } = useForm({
    defaultValues: {
      firstName: '',
      lastName: '',
      isChecked: false,
    },
    onSubmit: async ({ value }) => {
      // Обработка отправки формы
      console.log(value)
    },
  })

  return (
    <>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          handleSubmit()
        }}
      >
        <Field
          name="firstName"
          children={({ state, handleChange, handleBlur }) => (
            <TextInput
              defaultValue={state.value}
              onChange={(e) => handleChange(e.target.value)}
              onBlur={handleBlur}
              placeholder="Enter your name"
            />
          )}
        />
        <Field
          name="isChecked"
          children={({ state, handleChange, handleBlur }) => (
            <Checkbox
              onChange={(e) => handleChange(e.target.checked)}
              onBlur={handleBlur}
              checked={state.value}
            />
          )}
        />
      </form>
      <div>
        <pre>{JSON.stringify(state.values, null, 2)}</pre>
      </div>
    </>
  )
}
```

- Сначала мы используем хук `useForm` из TanStack и деструктурируем необходимые свойства. Этот шаг опционален — можно также использовать `const form = useForm()`, если предпочтительнее. Вывод типов TypeScript обеспечивает удобство работы в любом случае.
- Компонент `Field`, полученный из `useForm`, принимает несколько свойств, таких как `validators`. В этом примере мы сосредоточимся на двух основных: `name` и `children`.
  - Свойство `name` идентифицирует каждый `Field`, например, `firstName` в нашем примере.
  - Свойство `children` использует концепцию render props, позволяя интегрировать компоненты без лишних абстракций.
- Дизайн TanStack активно использует render props, предоставляя доступ к `children` внутри компонента `Field`. Этот подход полностью типобезопасен. При интеграции с компонентами Mantine, такими как `TextInput`, мы выборочно деструктурируем свойства, например `state.value`, `handleChange` и `handleBlur`. Такой выбор обусловлен небольшими различиями в типах между `TextInput` и `field`, который мы получаем в `children`.
- Следуя этим шагам, можно легко интегрировать компоненты Mantine с TanStack Form.
- Этот метод одинаково применим и к другим компонентам, например `Checkbox`, обеспечивая согласованную интеграцию для разных UI-элементов.

### Использование с Material UI

Процесс интеграции компонентов Material UI аналогичен. Вот пример с использованием TextField и Checkbox из Material UI:

```tsx
        <Field
            name="lastName"
            children={({ state, handleChange, handleBlur }) => {
              return (
                <TextField
                  id="filled-basic"
                  label="Filled"
                  variant="filled"
                  defaultValue={state.value}
                  onChange={(e) => handleChange(e.target.value)}
                  onBlur={handleBlur}
                  placeholder="Enter your last name"
                />
              );
            }}
          />

           <Field
            name="isMuiCheckBox"
            children={({ state, handleChange, handleBlur }) => {
              return (
                <MuiCheckbox
                  onChange={(e) => handleChange(e.target.checked)}
                  onBlur={handleBlur}
                  checked={state.value}
                />
              );
            }}
          />

```

- Подход к интеграции такой же, как и с Mantine.
- Основное отличие заключается в специфических свойствах и параметрах стилизации компонентов Material UI.
