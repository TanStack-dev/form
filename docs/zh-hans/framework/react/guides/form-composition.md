---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-12T04:10:42.863Z'
id: form-composition
title: 表单组合
---
# 表单组合 (Form Composition)

对 TanStack Form 的一个常见批评是其开箱即用的冗长性。虽然这对于教育目的可能有用——有助于强化对 API 的理解——但在生产用例中并不理想。

因此，虽然 `form.Field` 提供了 TanStack Form 最强大和灵活的使用方式，但我们提供了封装它的 API，以减少应用代码的冗长。

## 自定义表单钩子 (Custom Form Hooks)

组合表单最强大的方式是创建自定义表单钩子。这允许您创建一个适合应用需求的表单钩子，包括预绑定的自定义 UI 组件等。

在最基本的形式中，`createFormHook` 是一个接收 `fieldContext` 和 `formContext` 并返回 `useAppForm` 钩子的函数。

> 这个未定制的 `useAppForm` 钩子与 `useForm` 相同，但随着我们为 `createFormHook` 添加更多选项，这一点会很快改变。

```tsx
import { createFormHookContexts, createFormHook } from '@tanstack/react-form'

// 导出 useFieldContext 以供自定义组件使用
export const { fieldContext, formContext, useFieldContext } =
  createFormHookContexts()

const { useAppForm } = createFormHook({
  fieldContext,
  formContext,
  // 我们稍后会了解更多关于这些选项的内容
  fieldComponents: {},
  formComponents: {},
})

function App() {
  const form = useAppForm({
    // 支持所有 useForm 选项
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return <form.Field /> // ...
}
```

### 预绑定字段组件 (Pre-bound Field Components)

一旦搭建好这个基础结构，您就可以开始向表单钩子添加自定义字段和表单组件。

> 注意：`useFieldContext` 必须与自定义表单上下文中导出的相同

```tsx
import { useFieldContext } from './form-context.tsx'

export function TextField({ label }: { label: string }) {
  // `Field` 推断其 `value` 类型应为 `string`
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

然后您可以将此组件注册到表单钩子中。

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

并在表单中使用它：

```tsx
function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return (
    // 注意使用的是 `AppField` 而不是 `Field`；`AppField` 提供了所需的上下文
    <form.AppField
      name="firstName"
      children={(field) => <field.TextField label="First Name" />}
    />
  )
}
```

这不仅允许您复用共享组件的 UI，还保留了 TanStack Form 应有的类型安全：如果拼错 `name`，会得到一个 TypeScript 错误。

#### 关于性能的说明 (A note on performance)

虽然上下文是 React 生态系统中的一个有价值的工具，但许多用户合理地担心通过上下文提供响应式值会导致不必要的重新渲染。

> 不熟悉这个性能问题？[Mark Erikson 解释 Redux 如何解决这些问题的博客文章](https://blog.isquaredsoftware.com/2021/01/context-redux-differences/)是一个很好的起点。

虽然这是一个值得提出的问题，但对 TanStack Form 来说不是问题；通过上下文提供的值本身不是响应式的，而是具有响应式属性的静态类实例（[使用 TanStack Store 作为我们的信号实现来驱动整个过程](https://tanstack.com/store)）。

### 预绑定表单组件 (Pre-bound Form Components)

虽然 `form.AppField` 解决了许多字段样板和可复用性问题，但它并没有解决表单样板和可复用性问题。

特别是，能够共享 `form.Subscribe` 的实例，例如用于响应式表单提交按钮，是一个常见用例。

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
      // 注意 `AppForm` 组件包装器；`AppForm` 提供了所需的上下文
      <form.SubscribeButton label="Submit" />
    </form.AppForm>
  )
}
```

## 将大型表单拆分为小部分 (Breaking big forms into smaller pieces)

有时表单会变得非常大；有时情况就是这样。虽然 TanStack Form 能很好地支持大型表单，但处理数百或数千行代码的文件从来都不是一件有趣的事。

为了解决这个问题，我们支持使用 `withForm` 高阶组件将表单拆分为更小的部分。

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
  // 这些值仅用于类型检查，不在运行时使用
  // 这允许您从 `formOptions` 中 `...formOpts` 而无需重新声明选项
  defaultValues: {
    firstName: 'John',
    lastName: 'Doe',
  },
  // 可选，但会为 `render` 函数添加 props，除了 `form`
  props: {
    // 这些 props 也会设置为 `render` 函数的默认值
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

### `withForm` 常见问题 (FAQ)

> 为什么使用高阶组件而不是钩子？

虽然钩子是 React 的未来，但高阶组件仍然是组合的强大工具。特别是，`useForm` 的 API 使我们能够在不要求用户传递泛型的情况下实现强类型安全。

> 为什么我在 `render` 中收到关于钩子的 ESLint 错误？

ESLint 在函数的顶层寻找钩子，而 `render` 可能不会被识别为顶层组件，具体取决于您如何定义它。

```tsx
// 这会因钩子使用而导致 ESLint 错误
const ChildForm = withForm({
  // ...
  render: ({ form, title }) => {
    // ...
  },
})
```

```tsx
// 这样没问题
const ChildForm = withForm({
  // ...
  render: function Render({ form, title }) {
    // ...
  },
})
```

## 表单和字段组件的树摇 (Tree-shaking form and field components)

虽然上述示例非常适合入门，但对于某些用例（例如可能有数百个表单和字段组件的情况）并不理想。
特别是，您可能不希望将所有表单和字段组件包含在使用表单钩子的每个文件的包中。

为了解决这个问题，您可以将 `createFormHook` TanStack API 与 React 的 `lazy` 和 `Suspense` 组件混合使用：

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

这将在加载 `TextField` 组件时显示 Suspense 后备内容，加载完成后渲染表单。

## 整合所有内容 (Putting it all together)

现在我们已经介绍了创建自定义表单钩子的基础知识，让我们将所有内容整合到一个示例中。

```tsx
// /src/hooks/form.ts，用于整个应用
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

// /src/features/people/shared-form.ts，用于 `people` 功能
const formOpts = formOptions({
  defaultValues: {
    firstName: 'John',
    lastName: 'Doe',
  },
})

// /src/features/people/nested-form.ts，用于 `people` 页面
const ChildForm = withForm({
  ...formOpts,
  // 可选，但会为 `render` 函数添加 `form` 之外的 props
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

## API 使用指南 (API Usage Guidance)

以下图表可帮助您决定应使用哪些 API：

![](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/react_form_composability.svg)
