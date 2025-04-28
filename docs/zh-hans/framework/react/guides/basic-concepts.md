---
source-updated-at: '2025-03-26T16:17:59.000Z'
translation-updated-at: '2025-04-12T04:09:47.860Z'
id: basic-concepts
title: 基本概念
---

本页面介绍 `@tanstack/react-form` 库中使用的基本概念和术语。熟悉这些概念将帮助您更好地理解和使用该库。

## 表单选项 (Form Options)

您可以使用 `formOptions` 函数为表单创建可复用的配置选项：

```tsx
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## 表单实例 (Form Instance)

表单实例 (Form Instance) 是表示单个表单的对象，提供操作表单的方法和属性。您可以通过表单选项提供的 `useForm` 钩子创建表单实例。该钩子接受包含 `onSubmit` 函数的对象，该函数在表单提交时被调用。

```tsx
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // 处理表单数据
    console.log(value)
  },
})
```

也可以不使用 `formOptions`，直接通过独立的 `useForm` API 创建表单实例：

```tsx
const form = useForm({
  onSubmit: async ({ value }) => {
    // 处理表单数据
    console.log(value)
  },
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## 字段 (Field)

字段 (Field) 表示单个表单输入元素（如文本输入框或复选框）。通过表单实例提供的 `form.Field` 组件创建字段。该组件接受 `name` 属性（需匹配表单默认值中的键名）和 `children` 渲染函数属性（接收字段对象作为参数）。

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

每个字段都有独立的状态 (Field State)，包含当前值、验证状态、错误信息等元数据。可通过 `field.state` 属性访问：

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

字段有三种交互状态：当用户点击/聚焦时标记为 _"touched"_；值未改变时为 _"pristine"_；值改变后为 _"dirty"_。可通过以下标志位检查：

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![字段状态示意图](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

> **重要提示（对 React Hook Form 用户的说明）**：`TanStack/form` 中的 `isDirty` 标志与 RHF 中的同名标志不同。
> 在 RHF 中，当表单值与初始值不同时 `isDirty = true`。如果用户修改表单值后又改回默认值，RHF 中 `isDirty` 为 `false`，但在 `TanStack/form` 中仍为 `true`。
> 您可以通过 `form.options.defaultValues` 和 `field.options.defaultValue` 访问默认值，如需模拟 RHF 行为可自行实现 `isDefaultValue()` 辅助函数。

## 字段 API (Field API)

字段 API (Field API) 是创建字段时传递给渲染函数的对象，提供操作字段状态的方法：

```tsx
<input
  value={field.state.value}
  onBlur={field.handleBlur}
  onChange={(e) => field.handleChange(e.target.value)}
/>
```

## 验证 (Validation)

`@tanstack/react-form` 同时支持同步和异步验证。验证函数可通过 `form.Field` 组件的 `validators` 属性传递：

```tsx
<form.Field
  name="firstName"
  validators={{
    onChange: ({ value }) =>
      !value
        ? '必须填写名字'
        : value.length < 3
          ? '名字至少需要3个字符'
          : undefined,
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && '名字中不能包含"error"'
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

## 标准模式库验证 (Validation with Standard Schema Libraries)

除手动验证外，我们还支持 [Standard Schema](https://github.com/standard-schema/standard-schema) 规范。您可以使用任何实现该规范的库定义模式，并将其传递给表单或字段验证器。

支持的库包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```tsx
import { z } from 'zod'

const userSchema = z.object({
  age: z.number().gte(13, '必须年满13岁才能注册'),
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

`@tanstack/react-form` 提供多种订阅表单和字段状态变化的方式，最常用的是 `useStore(form.store)` 钩子和 `form.Subscribe` 组件。这些方法可优化渲染性能，仅在必要时更新组件：

```tsx
const firstName = useStore(form.store, (state) => state.values.firstName)
//...
<form.Subscribe
  selector={(state) => [state.canSubmit, state.isSubmitting]}
  children={([canSubmit, isSubmitting]) => (
    <button type="submit" disabled={!canSubmit}>
      {isSubmitting ? '提交中...' : '提交'}
    </button>
  )}
/>
```

请注意：虽然 `useStore` 钩子的 `selector` 属性是可选的，但强烈建议提供该参数，否则会导致不必要的重新渲染。

```tsx
// 正确用法
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
// 错误用法
const store = useStore(form.store)
```

注意：不推荐使用 `useField` 钩子实现响应式，该钩子设计用于在 `form.Field` 组件内部谨慎使用。建议改用 `useStore(form.store)`。

## 监听器 (Listeners)

`@tanstack/react-form` 允许您响应特定触发器并"监听"它们以执行副作用：

```tsx
<form.Field
  name="country"
  listeners={{
    onChange: ({ value }) => {
      console.log(`国家/地区变更为: ${value}, 正在重置省份`)
      form.setFieldValue('province', '')
    },
  }}
/>
```

更多信息请参阅 [监听器文档](./listeners.md)

## 数组字段 (Array Fields)

数组字段 (Array Fields) 用于管理表单中的值列表（如爱好列表）。通过 `form.Field` 组件设置 `mode="array"` 属性创建数组字段。

操作数组字段时，可使用 `pushValue`、`removeValue`、`swapValues` 和 `moveValue` 方法增删或交换数组元素：

```tsx
<form.Field
  name="hobbies"
  mode="array"
  children={(hobbiesField) => (
    <div>
      爱好
      <div>
        {!hobbiesField.state.value.length
          ? '暂无爱好'
          : hobbiesField.state.value.map((_, i) => (
              <div key={i}>
                <form.Field
                  name={`hobbies[${i}].name`}
                  children={(field) => {
                    return (
                      <div>
                        <label htmlFor={field.name}>名称：</label>
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
                          删除
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
                        <label htmlFor={field.name}>描述：</label>
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

以上是 `@tanstack/react-form` 库的基本概念和术语。理解这些概念将帮助您更高效地使用该库，轻松构建复杂表单。
