---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-29T23:15:12.395Z'
id: basic-concepts
title: 基础概念
---

本页面介绍 `@tanstack/svelte-form` 库中使用的基本概念和术语。熟悉这些概念将帮助您更好地理解和使用该库。

## 表单选项 (Form Options)

您可以使用 `formOptions` 函数创建表单选项，以便在多个表单之间共享配置。

示例：

```ts
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## 表单实例 (Form Instance)

表单实例 (Form Instance) 是表示单个表单的对象，提供操作表单的方法和属性。您可以使用 `createForm` 函数创建表单实例。该函数接收包含 `onSubmit` 函数的对象，该函数在表单提交时被调用。

```ts
const form = createForm(() => ({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // 处理表单数据
    console.log(value)
  },
}))
```

您也可以不使用 `formOptions` 创建表单实例：

```ts
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

字段 (Field) 表示单个表单输入元素，例如文本输入框或复选框。字段通过表单实例提供的 `form.Field` 组件创建。该组件接收 `name` 属性（需匹配表单默认值中的键名）和 `children` 属性（作为渲染函数的字段对象参数）。

示例：

```svelte
<form.Field name="firstName">
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onblur={field.handleBlur}
      oninput={(e) => field.handleChange(e.target.value)}
    />
  {/snippet}
</form.Field>
```

## 字段状态 (Field State)

每个字段都有其状态，包括当前值、验证状态、错误消息和其他元数据。您可以通过 `field.state` 属性访问字段状态。

示例：

```ts
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

有三种字段状态能有效反映用户交互行为：当用户点击/聚焦字段时为 _"已触碰 (touched)"_，用户未修改值时保持 _"原始状态 (pristine)"_，值被修改后变为 _"已修改 (dirty)"_。您可以通过以下标志位检查这些状态：

```ts
const { isTouched, isPristine, isDirty } = field.state.meta
```

![字段状态](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## 字段 API (Field API)

字段 API (Field API) 是创建字段时传递给渲染函数的对象，提供操作字段状态的方法。

示例：

```svelte
<input
  name={field.name}
  value={field.state.value}
  onblur={field.handleBlur}
  oninput={(e) => field.handleChange(e.target.value)}
/>
```

## 验证 (Validation)

`@tanstack/svelte-form` 内置同步和异步验证功能。验证函数可通过 `validators` 属性传递给 `form.Field` 组件。

示例：

```svelte
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
>
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onBlur={field.handleBlur}
      onInput={(e) => field.handleChange(e.target.value)}
    />
    <p>{field.state.meta.errors[0]}</p>
  {/snippet}
</form.Field>
```

## 标准模式库验证 (Validation with Standard Schema Libraries)

除手动验证外，我们还支持 [标准模式 (Standard Schema)](https://github.com/standard-schema/standard-schema) 规范。

您可以使用实现该规范的任何库定义模式，并将其传递给表单或字段验证器。

支持的库包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```svelte
<script>
  import { z } from 'zod'

  // ...
</script>

<form.Field
  name="firstName"
  validators={{
    onChange: z.string().min(3, '名字至少需要3个字符'),
    onChangeAsyncDebounceMs: 500,
    onChangeAsync: z.string().refine(
      async (value) => {
        await new Promise((resolve) => setTimeout(resolve, 1000))
        return !value.includes('error')
      },
      {
        message: '名字中不能包含"error"',
      },
    ),
  }}
>
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onBlur={field.handleBlur}
      onInput={(e) => field.handleChange(e.target.value)}
    />
    <p>{field.state.meta.errors[0]}</p>
  {/snippet}
</form.Field>
```

## 响应式 (Reactivity)

`@tanstack/svelte-form` 提供多种订阅表单和字段状态变化的方式，最突出的是 `form.useStore` 钩子和 `form.Subscribe` 组件。这些方法允许您通过仅在必要时更新组件来优化表单的渲染性能。

示例：

```svelte
<script>
  //...
  const firstName = form.useStore((state) => state.values.firstName)
</script>

<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
>
  {#snippet children(state)}
    <button type="submit" disabled={!state.canSubmit}>
      {state.isSubmitting ? '提交中...' : '提交'}
    </button>
  {/snippet}
</form.Subscribe>
```

## 数组字段 (Array Fields)

数组字段 (Array Fields) 允许您管理表单中的值列表，例如爱好列表。您可以通过设置 `mode="array"` 属性的 `form.Field` 组件创建数组字段。

操作数组字段时，可使用 `pushValue`、`removeValue`、`swapValues` 和 `moveValue` 方法来增删和交换数组中的值。

示例：

```svelte
<form.Field name="hobbies" mode="array">
  {#snippet children(hobbiesField)}
    <div>
      爱好
      <div>
        {#each hobbiesField.state.value as _, i}
            <div>
              <form.Field name={`hobbies[${i}].name`}>
                {#snippet children(field)}
                  <div>
                    <label for={field.name}>名称：</label>
                    <input
                      id={field.name}
                      name={field.name}
                      value={field.state.value}
                      onblur={field.handleBlur}
                      onchange={(e) => field.handleChange(e.target.value)}
                    />
                    <button
                      type="button"
                      onclick={() => hobbiesField.removeValue(i)}
                    >
                      删除
                    </button>
                  </div>
                {/snippet}
              </form.Field>
              <form.Field name={`hobbies[${i}].description`}>
                {#snippet children(field)}
                    <div>
                      <label for={field.name}>描述：</label>
                      <input
                        id={field.name}
                        name={field.name}
                        value={field.state.value}
                        onblur={field.handleBlur}
                        onchange={(e) => field.handleChange(e.target.value)}
                      />
                    </div>
                {/snippet}
              </form.Field>
            </div>
          {:else}
            暂无爱好
          {/each}
      </div>
      <button
        type="button"
        onclick={() =>
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
  {/snippet}
</form.Field>
```

以上是 `@tanstack/svelte-form` 库中使用的基本概念和术语。理解这些概念将帮助您更高效地使用该库，轻松创建复杂表单。
