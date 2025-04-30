---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T22:01:00.264Z'
id: form-composition
title: Композиция форм
---

# Композиция форм (Form Composition)

Распространённая критика TanStack Form — его избыточность "из коробки". Хотя это _может_ быть полезно для обучения (помогает лучше понять API), такой подход не идеален для продакшн-сценариев.

В результате, хотя `form.Field` обеспечивает наиболее мощное и гибкое использование TanStack Form, мы предоставляем API, которые оборачивают его и делают код приложения менее многословным.

## Пользовательские хуки форм (Custom Form Hooks)

Самый мощный способ композиции форм — создание пользовательских хуков. Это позволяет создать хук, адаптированный под нужды вашего приложения, включая предварительно связанные UI-компоненты и многое другое.

В самом простом случае `createFormHook` — это функция, которая принимает `fieldContext` и `formContext`, а возвращает хук `useAppForm`.

> Эта не настроенная версия хука `useAppForm` идентична `useForm`, но это быстро изменится, когда мы добавим больше опций в `createFormHook`.

```tsx
import { createFormHookContexts, createFormHook } from '@tanstack/react-form'

// экспортируем useFieldContext для использования в пользовательских компонентах
export const { fieldContext, formContext, useFieldContext } =
  createFormHookContexts()

const { useAppForm } = createFormHook({
  fieldContext,
  formContext,
  // Позже мы узнаем больше об этих опциях
  fieldComponents: {},
  formComponents: {},
})

function App() {
  const form = useAppForm({
    // Поддерживает все опции useForm
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return <form.Field /> // ...
}
```

### Предварительно связанные компоненты полей (Pre-bound Field Components)

После создания этой структуры можно начать добавлять пользовательские компоненты полей и форм в ваш хук.

> Важно: `useFieldContext` должен быть тем же самым, что экспортируется из вашего пользовательского контекста формы

```tsx
import { useFieldContext } from './form-context.tsx'

export function TextField({ label }: { label: string }) {
  // `Field` автоматически определяет, что тип `value` должен быть `string`
  const field = useFieldContext<string>()
  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}
```

Затем можно зарегистрировать этот компонент в вашем хуке формы.

```tsx
import { TextField } from './text-field.tsx'

const { useAppForm } = createFormHook({
  fieldContext,
  formContext,
  fieldComponents: {
    TextField,
  },
  formComponents: {},
})
```

И использовать его в форме:

```tsx
function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return (
    // Обратите внимание на `AppField` вместо `Field`; `AppField` предоставляет необходимый контекст
    <form.AppField
      name="firstName"
      children={(field) => <field.TextField label="First Name" />}
    />
  )
}
```

Это позволяет не только повторно использовать UI общих компонентов, но и сохраняет типобезопасность, ожидаемую от TanStack Form: опечатка в `name` вызовет ошибку TypeScript.

#### Замечание о производительности

Хотя контекст — ценный инструмент в экосистеме React, многие пользователи справедливо беспокоятся, что передача реактивных значений через контекст может вызвать лишние перерисовки.

> Не знакомы с этой проблемой производительности? [Статья Марка Эриксона, объясняющая, как Redux решает многие из этих проблем](https://blog.isquaredsoftware.com/2021/01/context-redux-differences/) — отличное место для начала.

Хотя это справедливое замечание, для TanStack Form это не проблема: значения, передаваемые через контекст, сами по себе не реактивны, а являются статическими экземплярами классов с реактивными свойствами ([используя TanStack Store как реализацию сигналов](https://tanstack.com/store)).

### Предварительно связанные компоненты форм (Pre-bound Form Components)

Хотя `form.AppField` решает многие проблемы с шаблонным кодом полей и их повторным использованием, он не решает проблему шаблонного кода _форм_ и их повторного использования.

В частности, возможность совместного использования экземпляров `form.Subscribe` для, например, реактивной кнопки отправки формы — распространённый сценарий.

```tsx
function SubscribeButton({ label }: { label: string }) {
  const form = useFormContext()
  return (
    <form.Subscribe selector={(state) => state.isSubmitting}>
      {(isSubmitting) => <button disabled={isSubmitting}>{label}</button>}
    </form.Subscribe>
  )
}

const { useAppForm, withForm } = createFormHook({
  fieldComponents: {},
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return (
    <form.AppForm>
      // Обратите внимание на обёртку `AppForm`; `AppForm` предоставляет
      необходимый контекст
      <form.SubscribeButton label="Submit" />
    </form.AppForm>
  )
}
```

## Разделение больших форм на части

Иногда формы становятся очень большими — так бывает. Хотя TanStack Form хорошо поддерживает большие формы, работать с файлами в сотни или тысячи строк кода никогда не бывает приятно.

Для решения этой проблемы мы поддерживаем разделение форм на меньшие части с использованием компонента высшего порядка `withForm`.

```tsx
const { useAppForm, withForm } = createFormHook({
  fieldComponents: {
    TextField,
  },
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

const ChildForm = withForm({
  // Эти значения используются только для проверки типов и не используются во время выполнения
  // Это позволяет использовать `...formOpts` из `formOptions` без необходимости переопределять опции
  defaultValues: {
    firstName: 'John',
    lastName: 'Doe',
  },
  // Опционально, но добавляет пропсы в функцию `render` помимо `form`
  props: {
    // Эти пропсы также устанавливаются как значения по умолчанию для функции `render`
    title: 'Child Form',
  },
  render: function Render({ form, title }) {
    return (
      <div>
        <p>{title}</p>
        <form.AppField
          name="firstName"
          children={(field) => <field.TextField label="First Name" />}
        />
        <form.AppForm>
          <form.SubscribeButton label="Submit" />
        </form.AppForm>
      </div>
    )
  },
})

function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return <ChildForm form={form} title={'Testing'} />
}
```

### FAQ по `withForm`

> Почему компонент высшего порядка (HOC), а не хук?

Хотя хуки — это будущее React, компоненты высшего порядка остаются мощным инструментом композиции. В частности, API `useForm` позволяет нам обеспечить строгую типобезопасность без необходимости передачи дженериков пользователями.

> Почему я получаю ошибки ESLint о хуках в `render`?

ESLint ищет хуки на верхнем уровне функции, и `render` может не распознаваться как компонент верхнего уровня, в зависимости от способа его определения.

```tsx
// Это вызовет ошибки ESLint при использовании хуков
const ChildForm = withForm({
  // ...
  render: ({ form, title }) => {
    // ...
  },
})
```

```tsx
// Это работает корректно
const ChildForm = withForm({
  // ...
  render: function Render({ form, title }) {
    // ...
  },
})
```

## Tree-shaking компонентов форм и полей

Хотя приведённые выше примеры отлично подходят для начала работы, они не идеальны для некоторых сценариев, где у вас могут быть сотни компонентов форм и полей.
В частности, вы можете не захотеть включать все компоненты форм и полей в сборку каждого файла, использующего ваш хук формы.

Для решения этой проблемы можно комбинировать API TanStack `createFormHook` с компонентами React `lazy` и `Suspense`:

```typescript
// src/hooks/form-context.ts
import { createFormHookContexts } from '@tanstack/react-form'

export const { fieldContext, useFieldContext, formContext, useFormContext } =
  createFormHookContexts()
```

```tsx
// src/components/text-field.tsx
import { useFieldContext } from '../hooks/form-context.tsx'

export default function TextField({ label }: { label: string }) {
  const field = useFieldContext<string>()

  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}
```

```tsx
// src/hooks/form.ts
import { lazy } from 'react'
import { createFormHook } from '@tanstack/react-form'

const TextField = lazy(() => import('../components/text-fields.tsx'))

const { useAppForm, withForm } = createFormHook({
  fieldContext,
  formContext,
  fieldComponents: {
    TextField,
  },
  formComponents: {},
})
```

```tsx
// src/App.tsx
import { Suspense } from 'react'
import { PeoplePage } from './features/people/page.tsx'

export default function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <PeopleForm />
    </Suspense>
  )
}
```

Это покажет fallback Suspense во время загрузки компонента `TextField`, а затем отрендерит форму после её загрузки.

## Собираем всё вместе

Теперь, когда мы рассмотрели основы создания пользовательских хуков форм, давайте соберём всё в одном примере.

```tsx
// /src/hooks/form.ts, для использования во всём приложении
const { fieldContext, useFieldContext, formContext, useFormContext } =
  createFormHookContexts()

function TextField({ label }: { label: string }) {
  const field = useFieldContext<string>()
  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}

function SubscribeButton({ label }: { label: string }) {
  const form = useFormContext()
  return (
    <form.Subscribe selector={(state) => state.isSubmitting}>
      {(isSubmitting) => <button disabled={isSubmitting}>{label}</button>}
    </form.Subscribe>
  )
}

const { useAppForm, withForm } = createFormHook({
  fieldComponents: {
    TextField,
  },
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

// /src/features/people/shared-form.ts, для использования в функциональности `people`
const formOpts = formOptions({
  defaultValues: {
    firstName: 'John',
    lastName: 'Doe',
  },
})

// /src/features/people/nested-form.ts, для использования на странице `people`
const ChildForm = withForm({
  ...formOpts,
  // Опционально, но добавляет пропсы в функцию `render` помимо `form`
  props: {
    title: 'Child Form',
  },
  render: ({ form, title }) => {
    return (
      <div>
        <p>{title}</p>
        <form.AppField
          name="firstName"
          children={(field) => <field.TextField label="First Name" />}
        />
        <form.AppForm>
          <form.SubscribeButton label="Submit" />
        </form.AppForm>
      </div>
    )
  },
})

// /src/features/people/page.ts
const Parent = () => {
  const form = useAppForm({
    ...formOpts,
  })

  return <ChildForm form={form} title={'Testing'} />
}
```

## Рекомендации по использованию API

Вот схема, которая поможет вам решить, какие API использовать:

![](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/react_form_composability.svg)
