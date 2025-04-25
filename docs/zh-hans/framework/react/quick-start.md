---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-12T04:08:52.516Z'
id: quick-start
title: 快速开始
---
TanStack Form 与你以往使用的大多数表单库不同。它专为大规模生产环境设计，注重类型安全、性能与组合能力，旨在提供无与伦比的开发者体验。

为此，我们制定了[库的使用哲学](/form/latest/docs/philosophy)，更重视可扩展性和长期开发者体验，而非简短可分享的代码片段。

以下是一个遵循多项最佳实践的示例表单，通过简短上手后，你就能快速开发出高复杂度的表单：

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { createFormHook, createFormHookContexts } from '@tanstack/react-form'
// 预绑定表单钩子事件的表单组件；详见我们的"表单组合"指南
import { TextField, NumberField, SubmitButton } from '~our-app/ui-library'
// 我们也支持 Valibot、ArkType 和其他标准模式库
import { z } from 'zod'

const { fieldContext, formContext } = createFormHookContexts()

// 允许我们将组件绑定到表单以保持类型安全，同时减少生产环境样板代码
// 定义一次即可在整个应用中生成一致的表单实例
const { useAppForm } = createFormHook({
  fieldComponents: {
    TextField,
    NumberField,
  },
  formComponents: {
    SubmitButton,
  },
  fieldContext,
  formContext,
})

const PeoplePage = () => {
  const form = useAppForm({
    defaultValues: {
      username: '',
      age: 0,
    },
    validators: {
      // 传入模式或函数进行验证
      onChange: z.object({
        username: z.string(),
        age: z.number().min(13),
      }),
    },
    onSubmit: ({ value }) => {
      // 处理表单数据
      alert(JSON.stringify(value, null, 2))
    },
  })

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault()
        form.handleSubmit()
      }}
    >
      <h1>Personal Information</h1>
      {/* 组件绑定到 `form` 和 `field` 以确保严格的类型安全 */}
      {/* 使用 `form.AppField` 渲染绑定到单个字段的组件 */}
      <form.AppField
        name="username"
        children={(field) => <field.TextField label="Full Name" />}
      />
      {/* "name" 属性若拼写错误会抛出 TypeScript 错误 */}
      <form.AppField
        name="age"
        children={(field) => <field.NumberField label="Age" />}
      />
      {/* `form.AppForm` 中的组件可访问表单上下文 */}
      <form.AppForm>
        <form.SubmitButton />
      </form.AppForm>
    </form>
  )
}

const rootElement = document.getElementById('root')!
ReactDOM.createRoot(rootElement).render(<PeoplePage />)
```

虽然我们通常建议长期使用 `createFormHook` 以减少样板代码，但我们也支持通过 `useForm` 和 `form.Field` 实现一次性组件和其他行为：

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { useForm } from '@tanstack/react-form'

const PeoplePage = () => {
  const form = useForm({
    defaultValues: {
      username: '',
      age: 0,
    },
    onSubmit: ({ value }) => {
      // 处理表单数据
      alert(JSON.stringify(value, null, 2))
    },
  })

  return (
    <form.Field
      name="age"
      validators={{
        // 可选择表单级或字段级验证器
        onChange: ({ value }) =>
          value > 13 ? undefined : '必须年满 13 岁',
      }}
      children={(field) => (
        <>
          <input
            name={field.name}
            value={field.state.value}
            onBlur={field.handleBlur}
            type="number"
            onChange={(e) => field.handleChange(e.target.valueAsNumber)}
          />
          {field.state.meta.errors.length ? (
            <em>{field.state.meta.errors.join(',')}</em>
          ) : null}
        </>
      )}
    />
  )
}

const rootElement = document.getElementById('root')!
ReactDOM.createRoot(rootElement).render(<PeoplePage />)
```

`useForm` 的所有属性都可在 `useAppForm` 中使用，`form.Field` 的所有属性也都适用于 `form.AppField`。
