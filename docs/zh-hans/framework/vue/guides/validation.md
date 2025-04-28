---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-12T04:11:25.487Z'
id: form-validation
title: 表单验证
---

TanStack Form 的核心功能之一是验证 (validation) 概念。TanStack Form 使验证具有高度可定制性：

- 您可以控制何时执行验证（在值变更时、输入时、失焦时、提交时...）
- 验证规则可以在字段 (field) 级别或表单 (form) 级别定义
- 验证可以是同步的或异步的（例如作为 API 调用的结果）

## 何时执行验证？

由您决定！`<Field />` 组件接受一些回调函数作为 props，例如 `onChange` 或 `onBlur`。这些回调会接收字段的当前值以及 `fieldAPI` 对象，以便您执行验证。如果发现验证错误，只需返回字符串形式的错误消息，它将出现在 `field.state.meta.errors` 中。

以下是一个示例：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="field.state.meta.errors">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

在上面的示例中，验证在每次按键时执行（`onChange`）。如果我们希望验证在字段失焦时执行，可以这样修改代码：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onBlur: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <!-- 我们始终需要实现 onChange，以便 TanStack Form 接收变更 -->
      <!-- 监听字段的 onBlur 事件 -->
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="field.state.meta.errors">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

因此，您可以通过实现所需的回调来控制验证的执行时机。您甚至可以在不同时间执行不同的验证逻辑：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
      onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <!-- 我们始终需要实现 onChange，以便 TanStack Form 接收变更 -->
      <!-- 监听字段的 onBlur 事件 -->
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="field.state.meta.errors">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

在上面的示例中，我们在同一字段的不同时间（每次按键和字段失焦时）验证不同的内容。由于 `field.state.meta.errors` 是一个数组，所有相关错误都会在给定时间显示。您还可以使用 `field.state.meta.errorMap` 根据验证执行时间（onChange、onBlur 等）获取错误。更多关于显示错误的信息如下。

## 显示错误

设置好验证后，您可以将错误从数组映射到 UI 中显示：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
      <em role="alert" v-if="field.state.meta.errors">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

或者使用 `errorMap` 属性访问特定错误：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
      <em role="alert" v-if="field.state.meta.errorMap['onChange']">{{
        field.state.meta.errorMap['onChange']
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

值得一提的是，我们的 `errors` 数组和 `errorMap` 与验证器返回的类型匹配。这意味着：

```vue
<form.Field
  name="age"
  :validators="{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }"
>
  <template v-slot="{ field }">
      <!-- ... -->
      <!-- errorMap.onChange 的类型是 `{isOldEnough: false} | undefined` -->
	  <!-- meta.errors 的类型是 `Array<{isOldEnough: false} | undefined>` -->
      <em v-if="!field.state.meta.errorMap['onChange']?.isOldEnough">The user is not old enough</em>
  </template>
</form.Field>
```

## 字段级验证与表单级验证

如上所示，每个 `<Field>` 通过 `onChange`、`onBlur` 等回调接受自己的验证规则。也可以通过向 `useForm()` 函数传递类似回调来在表单级别（而不是逐个字段）定义验证规则。

示例：

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    age: 0,
  },
  onSubmit: async ({ value }) => {
    console.log(value)
  },
  validators: {
    // 以与字段相同的方式向表单添加验证器
    onChange({ value }) {
      if (value.age < 13) {
        return 'Must be 13 or older to sign'
      }
      return undefined
    },
  },
})

// 订阅表单的错误映射，以便更新时重新渲染
// 或者，您可以使用 `form.Subscribe`
const formErrorMap = form.useStore((state) => state.errorMap)
</script>

<template>
  <!-- ... -->
  <div v-if="formErrorMap.onChange">
    <em role="alert">
      There was an error on the form: {{ formErrorMap.onChange }}
    </em>
  </div>
  <!-- ... -->
</template>
```

## 异步函数验证

虽然我们预计大多数验证将是同步的，但在许多情况下，通过网络调用或其他异步操作进行验证会很有用。

为此，我们提供了专用的 `onChangeAsync`、`onBlurAsync` 等方法：

```vue
<script setup lang="ts">
// ...

const onChangeAge = async ({ value }) => {
  await new Promise((resolve) => setTimeout(resolve, 1000))
  return value < 13 ? 'You must be 13 to make an account' : undefined
}
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChangeAsync: onChangeAge,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @input="
          (e) =>
            field.handleChange((e.target as HTMLInputElement).valueAsNumber)
        "
      />
      <em role="alert" v-if="field.state.meta.errors">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

同步和异步验证可以共存。例如，可以在同一字段上同时定义 `onBlur` 和 `onBlurAsync`：

```vue
<script setup lang="ts">
// ...

const onBlurAge = ({ value }) => (value < 0 ? 'Invalid value' : undefined)

const onBlurAgeAsync = async ({ value }) => {
  const currentAge = await fetchCurrentAgeOnProfile()
  return value < currentAge ? 'You can only increase the age' : undefined
}
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onBlur: onBlurAge,
      onBlurAsync: onBlurAgeAsync,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="
          (e) =>
            field.handleChange((e.target as HTMLInputElement).valueAsNumber)
        "
      />
      <em role="alert" v-if="field.state.meta.errors">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

同步验证方法（`onBlur`）首先运行，异步方法（`onBlurAsync`）仅在同步方法（`onBlur`）成功时运行。要改变此行为，请将 `asyncAlways` 选项设置为 `true`，这样无论同步方法的结果如何，异步方法都会运行。

### 内置防抖 (Debouncing)

虽然异步调用是验证数据库的好方法，但每次按键都运行网络请求是 DDOS 数据库的好方法。

相反，我们通过添加一个属性来提供一种简单的方法来防抖您的 `async` 调用：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :async-debounce-ms="500"
    :validators="{
      onChangeAsync: async ({ value }) => {
        // ...
      },
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

这将使每个异步调用延迟 500 毫秒执行。您甚至可以针对每个验证属性覆盖此设置：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :async-debounce-ms="500"
    :validators="{
      onChangeAsyncDebounceMs: 1500,
      onChangeAsync: async ({ value }) => {
        // ...
      },
      onBlurAsync: async ({ value }) => {
        // ...
      },
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

这将使 `onChangeAsync` 每 1500 毫秒运行一次，而 `onBlurAsync` 每 500 毫秒运行一次。

## 通过模式 (Schema) 库进行验证

虽然函数提供了更多灵活性和自定义验证的能力，但它们可能有些冗长。为了解决这个问题，有些库提供了基于模式 (schema) 的验证，使简写和类型严格的验证变得更加容易。您还可以为整个表单定义一个模式并将其传递到表单级别，错误将自动传播到字段。

### 标准模式 (Standard Schema) 库

TanStack Form 原生支持所有遵循 [Standard Schema 规范](https://github.com/standard-schema/standard-schema) 的库，最著名的是：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_注意：_ 请确保使用模式库的最新版本，旧版本可能尚不支持 Standard Schema。

要使用这些库中的模式，您可以像使用自定义函数一样将它们传递给 `validators` props：

```vue
<script setup lang="ts">
import { z } from 'zod'
// ...

const form = useForm({
  // ...
})
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: z.number().gte(13, 'You must be 13 to make an account'),
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

表单和字段级别的异步验证也受支持：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: z.number().gte(13, 'You must be 13 to make an account'),
      onChangeAsyncDebounceMs: 500,
      onChangeAsync: z.number().refine(
        async (value) => {
          const currentAge = await fetchCurrentAgeOnProfile()
          return value >= currentAge
        },
        {
          message: 'You can only increase the age',
        },
      ),
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## 阻止无效表单提交

`onChange`、`onBlur` 等回调在表单提交时也会运行，如果表单无效，提交将被阻止。

表单状态对象有一个 `canSubmit` 标志，当任何字段无效且表单已被触摸时，该标志为 false（`canSubmit` 在表单被触摸之前为 true，即使某些字段根据其 `onChange`/`onBlur` props "技术上" 无效）。

您可以通过 `form.Subscribe` 订阅它，并使用该值来执行诸如在表单无效时禁用提交按钮等操作（实际上，禁用的按钮不可访问，请改用 `aria-disabled`）。

```vue
<script setup lang="ts">
const form = useForm(/* ... */)
</script>

<template>
  <!-- ... -->

  <!-- 动态提交按钮 -->
  <form.Subscribe>
    <template v-slot="{ canSubmit, isSubmitting }">
      <button type="submit" :disabled="!canSubmit">
        {{ isSubmitting ? '...' : 'Submit' }}
      </button>
    </template>
  </form.Subscribe>
  <!-- ... -->
</template>
```
