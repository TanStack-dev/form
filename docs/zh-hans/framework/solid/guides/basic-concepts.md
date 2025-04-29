---
source-updated-at: '2025-03-14T21:54:05.000Z'
translation-updated-at: '2025-04-12T04:10:54.326Z'
id: basic-concepts
title: 基本概念
---

本页面介绍了 `@tanstack/solid-form` 库中使用的基本概念和术语。熟悉这些概念将帮助您更好地理解和使用该库。

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

表单实例是一个代表独立表单的对象，提供操作表单的方法和属性。您可以通过表单选项提供的 `createForm` 钩子创建实例，该钩子接收包含 `onSubmit` 函数的配置对象：

```tsx
const form = createForm(() => ({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // 处理表单数据
    console.log(value)
  },
}))
```

也可以直接使用独立的 `createForm` API 创建实例：

```tsx
const form = createForm<Person>(() => ({
  onSubmit: async ({ value }) => {
    // 处理表单数据
    console.log(value)
  },
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  },
}))
```

## 字段 (Field)

字段代表单个表单输入元素（如文本框或复选框），通过表单实例的 `form.Field` 组件创建。该组件接收与默认值匹配的 `name` 属性和接收字段对象的渲染函数：

```tsx
<form.Field
  name="firstName"
  children={(field) => (
    <input
      name={field().name}
      value={field().state.value}
      onBlur={field().handleBlur}
      onInput={(e) => field().handleChange(e.target.value)}
    />
  )}
/>
```

## 字段状态 (Field State)

每个字段都有包含当前值、验证状态和错误信息的状态对象，可通过 `field().state` 访问：

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field().state
```

字段有三种交互状态：获取焦点时为 _"touched"_，未修改时为 _"pristine"_，修改后为 _"dirty"_。可通过以下标志位检测：

```tsx
const { isTouched, isPristine, isDirty } = field().state.meta
```

![字段状态](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## 字段 API (Field API)

字段 API 是传递给渲染函数的对象，提供操作字段状态的方法：

```tsx
<input
  name={field().name}
  value={field().state.value}
  onBlur={field().handleBlur}
  onInput={(e) => field().handleChange(e.target.value)}
/>
```

## 验证 (Validation)

库内置同步/异步验证功能，可通过 `validators` 属性配置：

```tsx
<form.Field
  name="firstName"
  validators={{
    onChange: ({ value }) =>
      !value
        ? '必须填写名'
        : value.length < 3
          ? '名至少需要3个字符'
          : undefined,
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && '名中不能包含"error"'
    },
  }}
  children={(field) => (
    <>
      <input ... />
      <p>{field().state.meta.errors[0]}</p>
    </>
  )}
/>
```

## 标准模式库验证 (Validation with Standard Schema Libraries)

支持 [Standard Schema](https://github.com/standard-schema/standard-schema) 规范的验证库，包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```tsx
import { z } from 'zod'

// ...
;<form.Field
  name="firstName"
  validators={{
    onChange: z.string().min(3, '名至少需要3个字符'),
    onChangeAsyncDebounceMs: 500,
    onChangeAsync: z.string().refine(
      async (value) => {
        await new Promise((resolve) => setTimeout(resolve, 1000))
        return !value.includes('error')
      },
      { message: '名中不能包含"error"' },
    ),
  }}
/>
```

## 响应式 (Reactivity)

提供多种订阅状态变更的方式，包括 `form.useStore` 钩子和 `form.Subscribe` 组件：

```tsx
const firstName = form.useStore((state) => state.values.firstName)
//...
<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
  children={(state) => (
    <button type="submit" disabled={!state().canSubmit}>
      {state().isSubmitting ? '提交中...' : '提交'}
    </button>
  )}
/>
```

## 数组字段 (Array Fields)

通过 `mode="array"` 属性管理值列表，支持 `pushValue`/`removeValue` 等方法：

```tsx
<form.Field
  name="hobbies"
  mode="array"
  children={(hobbiesField) => (
    <div>
      爱好
      <div>
        <Show when={hobbiesField().state.value.length > 0} fallback={'暂无爱好'}>
          <Index each={hobbiesField().state.value}>
            {(_, i) => (
              <div>
                <form.Field name={`hobbies[${i}].name`} ... />
                <form.Field name={`hobbies[${i}].description`} ... />
              </div>
            )}
          </Index>
        </Show>
      </div>
      <button
        type="button"
        onClick={() => hobbiesField().pushValue({ name: '', description: '' })}
      >
        添加爱好
      </button>
    </div>
  )}
/>
```

以上是 `@tanstack/solid-form` 库的核心概念和术语，掌握这些内容将帮助您高效构建复杂表单。
