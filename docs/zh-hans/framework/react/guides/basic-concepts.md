---
source-updated-at: '2025-04-16T08:45:06.000Z'
translation-updated-at: '2025-04-29T23:15:30.242Z'
id: basic-concepts
title: 基础概念
---

本页介绍 `@tanstack/react-form` 库中使用的基本概念和术语。熟悉这些概念将帮助您更好地理解和使用该库。

## 表单选项 (Form Options)

您可以通过 `formOptions` 函数创建表单选项，以便在多个表单之间共享配置。

示例：

```tsx
interface User {
  firstName: string
  lastName: string
  hobbies: Array<string>
}
const defaultUser: User = { firstName: '', lastName: '', hobbies: [] }

const formOpts = formOptions({
  defaultValues: defaultUser,
})
```

## 表单实例 (Form Instance)

表单实例是一个表示单个表单的对象，提供了操作表单的方法和属性。您可以通过表单选项提供的 `useForm` 钩子创建表单实例。该钩子接受一个包含 `onSubmit` 函数的对象，该函数在表单提交时被调用。

```tsx
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // 处理表单数据
    console.log(value)
  },
})
```

您也可以不使用 `formOptions`，直接通过独立的 `useForm` API 创建表单实例：

```tsx
interface User {
  firstName: string
  lastName: string
  hobbies: Array<string>
}
const defaultUser: User = { firstName: '', lastName: '', hobbies: [] }

const form = useForm({
  defaultValues: defaultUser,
  onSubmit: async ({ value }) => {
    // 处理表单数据
    console.log(value)
  },
})
```

## 字段 (Field)

字段表示单个表单输入元素，例如文本输入框或复选框。字段通过表单实例提供的 `form.Field` 组件创建。该组件接受一个 `name` 属性，应与表单默认值中的键匹配。它还接受一个 `children` 属性，这是一个渲染函数，接收字段对象作为参数。

示例：

```tsx
<form.Field
  name="firstName"
  children={(field) => (
    <>
      <input
        value={field.state.value}
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.value)}
      />
      <FieldInfo field={field} />
    </>
  )}
/>
```

## 字段状态 (Field State)

每个字段都有自己的状态，包括当前值、验证状态、错误消息和其他元数据。您可以通过 `field.state` 属性访问字段状态。

示例：

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

有三种字段状态可用于观察用户交互行为：当用户点击/聚焦字段时为 _"touched"_，用户未修改字段值为 _"pristine"_，修改字段值后为 _"dirty"_。您可以通过 `isTouched`、`isPristine` 和 `isDirty` 标志检查这些状态，如下所示。

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![字段状态](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

> **对于从 `React Hook Form` 迁移的用户的重要提示**：`TanStack/form` 中的 `isDirty` 标志与 RHF 中的同名标志不同。
> 在 RHF 中，当表单值与原始值不同时 `isDirty = true`。如果用户修改表单值后又改回与默认值相同的值，RHF 中的 `isDirty` 将为 `false`，但 `TanStack/form` 中仍为 `true`。
> `TanStack/form` 在表单和字段级别都暴露了默认值 (`form.options.defaultValues`, `field.options.defaultValue`)，因此如果需要模拟 RHF 的行为，您可以编写自己的 `isDefaultValue()` 辅助函数。

## 字段 API (Field API)

字段 API 是创建字段时传递给渲染函数的对象，提供了操作字段状态的方法。

示例：

```tsx
<input
  value={field.state.value}
  onBlur={field.handleBlur}
  onChange={(e) => field.handleChange(e.target.value)}
/>
```

## 验证 (Validation)

`@tanstack/react-form` 内置支持同步和异步验证。验证函数可以通过 `validators` 属性传递给 `form.Field` 组件。

示例：

```tsx
<form.Field
  name="firstName"
  validators={{
    onChange: ({ value }) =>
      !value
        ? '必须填写名字'
        : value.length < 3
          ? '名字至少需要 3 个字符'
          : undefined,
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && '名字中不能包含 "error"'
    },
  }}
  children={(field) => (
    <>
      <input
        value={field.state.value}
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.value)}
      />
      <FieldInfo field={field} />
    </>
  )}
/>
```

## 使用标准模式库验证 (Validation with Standard Schema Libraries)

除了手动编写验证逻辑外，我们还支持 [Standard Schema](https://github.com/standard-schema/standard-schema) 规范。

您可以使用任何实现该规范的库定义模式，并将其传递给表单或字段验证器。

支持的库包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```tsx
import { z } from 'zod'

const userSchema = z.object({
  age: z.number().gte(13, '必须年满 13 岁才能创建账户'),
})

function App() {
  const form = useForm({
    defaultValues: {
      age: 0,
    },
    validators: {
      onChange: userSchema,
    },
  })
  return (
    <div>
      <form.Field
        name="age"
        children={(field) => {
          return <>{/* ... */}</>
        }}
      />
    </div>
  )
}
```

## 响应式 (Reactivity)

`@tanstack/react-form` 提供了多种订阅表单和字段状态变化的方式，最突出的是 `useStore(form.store)` 钩子和 `form.Subscribe` 组件。这些方法允许您通过仅在必要时更新组件来优化表单的渲染性能。

示例：

```tsx
const firstName = useStore(form.store, (state) => state.values.firstName)
//...
<form.Subscribe
  selector={(state) => [state.canSubmit, state.isSubmitting]}
  children={([canSubmit, isSubmitting]) => (
    <button type="submit" disabled={!canSubmit}>
      {isSubmitting ? '...' : '提交'}
    </button>
  )}
/>
```

需要注意的是，虽然 `useStore` 钩子的 `selector` 属性是可选的，但强烈建议提供该属性，因为省略它会导致不必要的重新渲染。

```tsx
// 正确用法
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
// 错误用法
const store = useStore(form.store)
```

注意：不推荐使用 `useField` 钩子来实现响应式，因为它设计用于在 `form.Field` 组件内谨慎使用。您可能需要改用 `useStore(form.store)`。

## 监听器 (Listeners)

`@tanstack/react-form` 允许您响应特定触发器并"监听"它们以派发副作用。

示例：

```tsx
<form.Field
  name="country"
  listeners={{
    onChange: ({ value }) => {
      console.log(`国家已更改为: ${value}, 重置省份`)
      form.setFieldValue('province', '')
    },
  }}
/>
```

更多信息请参阅 [监听器](./listeners.md)

## 数组字段 (Array Fields)

数组字段允许您管理表单中的值列表，例如爱好列表。您可以通过设置 `mode="array"` 属性的 `form.Field` 组件创建数组字段。

处理数组字段时，可以使用 `pushValue`、`removeValue`、`swapValues` 和 `moveValue` 方法来添加、删除和交换数组中的值。

示例：

```tsx
<form.Field
  name="hobbies"
  mode="array"
  children={(hobbiesField) => (
    <div>
      爱好
      <div>
        {!hobbiesField.state.value.length
          ? '未找到爱好。'
          : hobbiesField.state.value.map((_, i) => (
              <div key={i}>
                <form.Field
                  name={`hobbies[${i}].name`}
                  children={(field) => {
                    return (
                      <div>
                        <label htmlFor={field.name}>名称:</label>
                        <input
                          id={field.name}
                          name={field.name}
                          value={field.state.value}
                          onBlur={field.handleBlur}
                          onChange={(e) => field.handleChange(e.target.value)}
                        />
                        <button
                          type="button"
                          onClick={() => hobbiesField.removeValue(i)}
                        >
                          X
                        </button>
                        <FieldInfo field={field} />
                      </div>
                    )
                  }}
                />
                <form.Field
                  name={`hobbies[${i}].description`}
                  children={(field) => {
                    return (
                      <div>
                        <label htmlFor={field.name}>描述:</label>
                        <input
                          id={field.name}
                          name={field.name}
                          value={field.state.value}
                          onBlur={field.handleBlur}
                          onChange={(e) => field.handleChange(e.target.value)}
                        />
                        <FieldInfo field={field} />
                      </div>
                    )
                  }}
                />
              </div>
            ))}
      </div>
      <button
        type="button"
        onClick={() =>
          hobbiesField.pushValue({
            name: '',
            description: '',
            yearsOfExperience: 0,
          })
        }
      >
        添加爱好
      </button>
    </div>
  )}
/>
```

## 重置按钮 (Reset Buttons)

当将 `<button type="reset">` 与 TanStack Form 的 `form.reset()` 结合使用时，需要阻止默认的 HTML 重置行为，以避免表单元素（特别是 `<select>` 元素）意外重置为其初始 HTML 值。
在按钮的 `onClick` 处理程序中使用 `event.preventDefault()` 来阻止原生表单重置。

示例：

```tsx
<button
  type="reset"
  onClick={(event) => {
    event.preventDefault()
    form.reset()
  }}
>
  重置
</button>
```

或者，您可以使用 `<button type="button">` 来阻止原生 HTML 重置。

```tsx
<button
  type="button"
  onClick={() => {
    form.reset()
  }}
>
  重置
</button>
```

以上是 `@tanstack/react-form` 库中使用的基本概念和术语。理解这些概念将帮助您更高效地使用该库，轻松创建复杂的表单。
