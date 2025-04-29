---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-29T23:23:20.571Z'
id: quick-start
title: 快速开始
---

TanStack Form 与您以往使用的大多数表单库不同。它专为大规模生产环境设计，注重类型安全、性能优化和组合能力，旨在提供无与伦比的开发体验。

为此，我们制定了[关于该库使用理念](/form/latest/docs/philosophy)，更重视可扩展性和长期开发体验，而非简短可分享的代码片段。

以下示例展示了遵循我们多项最佳实践的表单实现，通过短暂学习后，您就能快速开发出高度复杂的表单：

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { createFormHook, createFormHookContexts } from '@tanstack/react-form'
// 预绑定表单事件的表单组件；详见我们的"表单组合"指南
import { TextField, NumberField, SubmitButton } from '~our-app/ui-library'
// 同时支持 Valibot、ArkType 及其他标准模式库
import { z } from 'zod'

const { fieldContext, formContext } = createFormHookContexts()

// 允许绑定组件到表单，保持类型安全的同时减少生产环境样板代码
// 只需定义一次，即可在应用中生成一致的表单实例
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
      {/* 组件绑定到 `form` 和 `field` 确保严格的类型安全 */}
      {/* 使用 `form.AppField` 渲染绑定到单个字段的组件 */}
      <form.AppField
        name="username"
        children={(field) => <field.TextField label="Full Name" />}
      />
      {/* 若拼写错误，"name"属性会抛出 TypeScript 错误 */}
      <form.AppField
        name="age"
        children={(field) => <field.NumberField label="Age" />}
      />
      {/* `form.AppForm`中的组件可访问表单上下文 */}
      <form.AppForm>
        <form.SubmitButton />
      </form.AppForm>
    </form>
  )
}

const rootElement = document.getElementById('root')!
ReactDOM.createRoot(rootElement).render(<PeoplePage />)
```

虽然我们通常建议长期使用 `createFormHook` 来减少样板代码，但我们也支持通过 `useForm` 和 `form.Field` 实现一次性组件和其他行为：

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
        onChange: ({ value }) => (value > 13 ? undefined : '必须年满13周岁'),
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
          {!field.state.meta.isValid && (
            <em>{field.state.meta.errors.join(',')}</em>
          )}
        </>
      )}
    />
  )
}

const rootElement = document.getElementById('root')!
ReactDOM.createRoot(rootElement).render(<PeoplePage />)
```

`useForm` 的所有属性都可在 `useAppForm` 中使用，`form.Field` 的所有属性也都适用于 `form.AppField`。
