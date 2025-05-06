---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:17:13.292Z'
id: submission-handling
title: Submission handling
---

## 向提交处理传递额外数据

您可能需要处理多种提交行为类型，例如返回其他页面或停留在当前表单。可以通过指定 `onSubmitMeta` 属性实现这一需求，这些元数据将被传递到 `onSubmit` 函数中。

> 注意：如果调用 `form.handleSubmit()` 时未提供元数据，将使用指定的默认值。

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null
}

// 调用 form.handleSubmit() 时不强制要求传递元数据
// 指定未传递元数据时使用的默认值
const defaultMeta: FormMeta = {
  submitAction: null,
}

const form = useForm({
  defaultValues: {
    data: '',
  },
  // 定义提交时期望接收的元数据值
  onSubmitMeta: defaultMeta,
  onSubmit: async ({ value, meta }) => {
    // 处理通过 handleSubmit 传递的值
    console.log(`Selected action - ${meta.submitAction}`, value)
  },
})
</script>

<template>
  <form @submit.prevent.stop="">
    <button
      type="submit"
      @click="
        () => {
          // 覆盖 onSubmitMeta 中指定的默认值
          form.handleSubmit({ submitAction: 'continue' })
        }
      "
    >
      提交并继续
    </button>
    <button
      type="submit"
      @click="() => form.handleSubmit({ submitAction: 'backToMenu' })"
    >
      提交并返回菜单
    </button>
  </form>
</template>
```

## 使用标准模式转换数据

虽然 Tanstack Form 提供了[标准模式支持 (Standard Schema)](./validation.md) 进行验证，但不会保留模式输出的数据。

传递给 `onSubmit` 函数的值始终是输入数据。要获取标准模式的输出数据，需要在 `onSubmit` 函数中进行解析：

```vue
<script setup lang="ts">
import { z } from 'zod'
import { useForm } from '@tanstack/vue-form'

const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form 使用标准模式的输入类型
const defaultValues: z.input<typeof schema> = {
  age: '13',
}

const form = useForm({
  defaultValues,
  validators: {
    onChange: schema,
  },
  onSubmit: ({ value }) => {
    const inputAge: string = value.age
    // 通过模式解析获取转换后的值
    const result = schema.parse(value)
    const outputAge: number = result.age
  },
})
</script>
<template>
  <!-- ... -->
</template>
```
