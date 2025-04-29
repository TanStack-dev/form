---
source-updated-at: '2025-04-16T14:15:06.000Z'
translation-updated-at: '2025-04-29T23:16:08.399Z'
id: form-validation
title: 表单验证
---

TanStack Form 的核心功能之一是验证 (validation) 概念。TanStack Form 提供了高度可定制的验证机制：

- 您可以控制验证时机（值变更时、输入时、失焦时、提交时等）
- 验证规则可以在字段级别或表单级别定义
- 验证可以是同步或异步的（例如通过 API 调用返回结果）

## 验证触发时机

完全由您决定！`<Field />` 组件接受一些回调函数作为 props，如 `onChange` 或 `onBlur`。这些回调会接收字段当前值和 `fieldAPI` 对象，以便执行验证。如果发现验证错误，只需返回字符串类型的错误信息，该信息将通过 `field.state.meta.errors` 获取。

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

上例中，验证在每次按键时触发（`onChange`）。如果我们想让验证在字段失焦时执行，可以修改代码如下：

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
      <!-- 始终需要实现 onChange 以便 TanStack Form 接收变更 -->
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

通过实现不同的回调函数，您可以控制验证时机。甚至可以在不同时间执行不同的验证逻辑：

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
      <!-- 始终需要实现 onChange 以便 TanStack Form 接收变更 -->
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

上例中，我们在同一字段的不同时机（每次按键和失焦时）执行不同验证。由于 `field.state.meta.errors` 是数组，会显示当前所有相关错误。您也可以使用 `field.state.meta.errorMap` 根据验证时机（onChange、onBlur 等）获取错误信息。更多错误显示方式见下文。

## 错误显示

设置验证后，您可以将错误数组映射到 UI 中显示：

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

或使用 `errorMap` 属性访问特定错误：

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

值得注意的是，我们的 `errors` 数组和 `errorMap` 类型与验证器返回类型匹配。这意味着：

```vue
<form.Field
  name="age"
  :validators="{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }"
>
  <template v-slot="{ field }">
      <!-- ... -->
      <!-- errorMap.onChange 类型为 `{isOldEnough: false} | undefined` -->
	  <!-- meta.errors 类型为 `Array<{isOldEnough: false} | undefined>` -->
      <em v-if="!field.state.meta.errorMap['onChange']?.isOldEnough">The user is not old enough</em>
  </template>
</form.Field>
```

## 字段级验证 vs 表单级验证

如上所示，每个 `<Field>` 通过 `onChange`、`onBlur` 等回调接受自己的验证规则。也可以通过向 `useForm()` 函数传递类似回调来定义表单级验证规则（而非逐个字段定义）。

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

// 订阅表单的错误映射以便更新时重新渲染
// 或者可以使用 `form.Subscribe`
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

虽然大多数验证可能是同步的，但很多情况下需要通过网络调用或其他异步操作进行验证。

为此，我们提供了专门的 `onChangeAsync`、`onBlurAsync` 等方法：

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

同步验证方法（`onBlur`）会先执行，只有同步验证成功后才会执行异步方法（`onBlurAsync`）。要改变此行为，可将 `asyncAlways` 选项设为 `true`，这样无论同步方法结果如何都会执行异步方法。

### 内置防抖

虽然异步调用是验证数据库的有效方式，但每次按键都发送网络请求可能会导致数据库遭受 DDoS 攻击。

为此，我们提供了简单的防抖方法，只需添加一个属性：

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

这将使所有异步调用延迟 500ms 执行。您还可以为每个验证属性单独覆盖此设置：

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

这将使 `onChangeAsync` 每 1500ms 执行一次，而 `onBlurAsync` 仍保持 500ms 间隔。

## 通过模式库验证

虽然函数提供了更灵活的验证方式，但可能略显冗长。为解决此问题，有些库提供了基于模式 (schema) 的验证，可以大幅简化类型严格的验证。您还可以为整个表单定义单一模式并传递给表单级别，错误会自动传播到各字段。

### 标准模式库

TanStack Form 原生支持所有遵循 [Standard Schema 规范](https://github.com/standard-schema/standard-schema) 的库，最著名的包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_注意：_ 请确保使用模式库的最新版本，旧版本可能尚未支持 Standard Schema。

您可以将这些库的模式传递给 `validators` props，就像使用自定义函数一样：

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

如果需要更多控制，可以将 Standard Schema 与回调函数结合使用：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value, fieldApi }) => {
        const errors = fieldApi.parseValueWithSchema(
          z.number().gte(13, 'You must be 13 to make an account'),
        )

        if (errors) return errors

        // 继续您的验证逻辑
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

## 阻止无效表单提交

提交表单时也会运行 `onChange`、`onBlur` 等回调，如果表单无效则会阻止提交。

表单状态对象有一个 `canSubmit` 标志，当任何字段无效且表单已被触碰时该标志为 false（即使某些字段根据其 `onChange`/`onBlur` props "技术上" 无效，只要表单未被触碰，`canSubmit` 仍为 true）。

您可以通过 `form.Subscribe` 订阅该值，例如用于在表单无效时禁用提交按钮（实践中，禁用按钮不利于可访问性，应使用 `aria-disabled`）。

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
