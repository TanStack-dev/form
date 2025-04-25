---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-12T04:10:42.406Z'
id: basic-concepts
title: 基本概念
---
本页介绍 `@tanstack/vue-form` 库中使用的基本概念和术语。熟悉这些概念将帮助您更好地理解和使用该库。

## 表单配置项 (Form Options)

您可以通过 `formOptions` 函数创建表单配置项，以便在多个表单之间共享配置。

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

表单实例是一个表示独立表单的对象，提供操作表单的方法和属性。您可以使用 `useForm` 函数创建表单实例。该函数接收包含 `onSubmit` 函数的对象，该函数在表单提交时被调用。

```js
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // 处理表单数据
    console.log(value)
  },
})
```

您也可以不使用 `formOptions`，直接通过独立的 `useForm` API 创建表单实例：

```ts
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

字段表示单个表单输入元素，例如文本输入框或复选框。字段通过表单实例提供的 `form.Field` 组件创建。该组件接收 `name` 属性，应与表单默认值中的键名匹配。同时接受通过 `v-slot` 指令定义的作用域插槽，该插槽接收 `field` 对象作为参数。

示例：

```vue
<template>
  <!-- ... -->
  <form.Field name="fullName">
    <template v-slot="{ field }">
      <input
        :name="field.name"
        :value="field.state.value"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange(e.target.value)"
      />
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## 字段状态 (Field State)

每个字段都有自己的状态，包括当前值、验证状态、错误消息和其他元数据。您可以通过 `field.state` 属性访问字段状态。

示例：

```js
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

有三种字段状态对观察用户交互非常有用：当用户点击/聚焦字段时为 _"已触碰 (touched)"_，用户未修改值时保持 _"原始状态 (pristine)"_，值被修改后变为 _"已修改 (dirty)"_。您可以通过 `isTouched`、`isPristine` 和 `isDirty` 标志检查这些状态，如下所示。

```js
const { isTouched, isPristine, isDirty } = field.state.meta
```

![字段状态](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## 字段 API (Field API)

字段 API 是通过 `v-slot` 指令提供的作用域插槽对象。该插槽接收名为 `field` 的参数，提供操作字段状态的方法和属性。

示例：

```vue
<template v-slot="{ field }">
  <input
    :name="field.name"
    :value="field.state.value"
    @blur="field.handleBlur"
    @input="(e) => field.handleChange(e.target.value)"
  />
</template>
```

## 验证 (Validation)

`@tanstack/vue-form` 开箱即用地支持同步和异步验证。验证函数可以通过 `validators` 属性传递给 `form.Field` 组件。

示例：

```vue
<template>
    <!-- ... -->
    <form.Field
        name="firstName"
        :validators="{
            onChange: ({ value }) =>
                !value
                    ? `姓名不能为空`
                    : value.length < 3
                        ? `姓名至少需要 3 个字符`
                        : undefined,
            onChangeAsync: async ({ value }) => {
                await new Promise((resolve) => setTimeout(resolve, 1000))
                return value.includes('error') && '姓名中不能包含 "error"'
        },
    }"
    >
        <template v-slot="{ field }">
            <input
                :value="field.state.value"
                @input="(e) => field.handleChange(e.target.value)"
                @blur="field.handleBlur"
            />
            <FieldInfo :field="field" />
        </template>
    </form.Field>
    <!-- ... -->
</template>
```

## 标准模式库验证 (Validation with Standard Schema Libraries)

除了手动编写的验证选项外，我们还支持 [Standard Schema](https://github.com/standard-schema/standard-schema) 规范。

您可以使用任何实现该规范的库定义模式，并将其传递给表单或字段验证器。

支持的库包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'
import { z } from 'zod'

const form = useForm({
  // ...
})

const onChangeFirstName = z.string().refine(
  async (value) => {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return !value.includes('error')
  },
  {
    message: "姓名中不能包含 'error'",
  },
)
</script>

<template>
  <!-- ... -->
  <form.Field
    name="firstName"
    :validators="{
      onChange: z.string().min(3, '姓名至少需要 3 个字符'),
      onChangeAsyncDebounceMs: 500,
      onChangeAsync: onChangeFirstName,
    }"
  >
    <template v-slot="{ field, state }">
      <label :htmlFor="field.name">姓名:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).value)"
        @blur="field.handleBlur"
      />
      <FieldInfo :state="state" />
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## 响应式 (Reactivity)

`@tanstack/vue-form` 提供多种订阅表单和字段状态变化的方式，最显著的是 `form.useStore` 方法和 `form.Subscribe` 组件。这些方法允许您通过仅在必要时更新组件来优化表单的渲染性能。

示例：

```vue
<script setup lang="ts">
// ...
const firstName = form.useStore((state) => state.values.firstName)
</script>

<template>
  <!-- ... -->
  <form.Subscribe>
    <template v-slot="{ canSubmit, isSubmitting }">
      <button type="submit" :disabled="!canSubmit">
        {{ isSubmitting ? '...' : '提交' }}
      </button>
    </template>
  </form.Subscribe>
  <!-- ... -->
</template>
```

## 监听器 (Listeners)

`@tanstack/vue-form` 允许您响应特定触发器并"监听"它们以派发副作用。

示例：

```vue
<template>
  <form.Field
    name="country"
    :listeners="{
      onChange: ({ value }) => {
        console.log(`国家已更改为: ${value}, 重置省份`)
        form.setFieldValue('province', '')
      },
    }"
  >
    <template v-slot="{ field }">
      <input
        :value="field.state.value"
        @input="(e) => field.handleChange(e.target.value)"
      />
    </template>
  </form.Field>
</template>
```

更多信息请参阅 [监听器](./listeners.md)

注意：不推荐使用 `form.useField` 方法实现响应式，因为该方法设计用于在 `form.Field` 组件中谨慎使用。您可能需要改用 `form.useStore`。

## 数组字段 (Array Fields)

数组字段允许您管理表单中的值列表，例如爱好列表。您可以通过带有 `mode="array"` 属性的 `form.Field` 组件创建数组字段。

处理数组字段时，可以使用字段的 `pushValue`、`removeValue`、`swapValues` 和 `moveValue` 方法来添加、删除和交换数组中的值。

示例：

```vue
<template>
  <form @submit.prevent.stop="form.handleSubmit">
    <form.Field name="hobbies" mode="array">
      <template v-slot="{ field: hobbiesField }">
        <div>
          爱好
          <div>
            <div
              v-if="
                Array.isArray(hobbiesField.state.value) &&
                !hobbiesField.state.value.length
              "
            >
              暂无爱好
            </div>
            <div v-else>
              <div v-for="(_, i) in hobbiesField.state.value" :key="i">
                <form.Field :name="`hobbies[${i}].name`">
                  <template v-slot="{ field }">
                    <div>
                      <label :for="field.name">名称:</label>
                      <input
                        :id="field.name"
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="(e) => field.handleChange(e.target.value)"
                      />
                      <button
                        type="button"
                        @click="hobbiesField.removeValue(i)"
                      >
                        X
                      </button>
                      <FieldInfo :field="field" />
                    </div>
                  </template>
                </form.Field>
                <form.Field :name="`hobbies[${i}].description`">
                  <template v-slot="{ field }">
                    <div>
                      <label :for="field.name">描述:</label>
                      <input
                        :id="field.name"
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="(e) => field.handleChange(e.target.value)"
                      />
                      <FieldInfo :field="field" />
                    </div>
                  </template>
                </form.Field>
              </div>
            </div>
            <button
              type="button"
              @click="
                hobbiesField.pushValue({
                  name: '',
                  description: '',
                  yearsOfExperience: 0,
                })
              "
            >
              添加爱好
            </button>
          </div>
        </div>
      </template>
    </form.Field>
  </form>
</template>
```

以上是 `@tanstack/vue-form` 库中使用的基本概念和术语。理解这些概念将帮助您更高效地使用该库，轻松创建复杂表单。
