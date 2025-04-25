---
source-updated-at: '2025-03-31T17:05:18.000Z'
translation-updated-at: '2025-04-12T04:08:22.956Z'
id: quick-start
title: 快速开始
---
## 快速开始

使用 TanStack Form 的最低要求是创建一个表单并添加一个字段。请注意，当前示例尚未包含任何验证或错误处理功能。

```vue
<!-- App.vue -->
<script setup>
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    fullName: '',
  },
  onSubmit: async ({ value }) => {
    // 处理表单数据
    console.log(value)
  },
})
</script>

<template>
  <div>
    <form @submit.prevent.stop="form.handleSubmit">
      <div>
        <form.Field name="fullName">
          <template v-slot="{ field }">
            <input
              :name="field.name"
              :value="field.state.value"
              @blur="field.handleBlur"
              @input="(e) => field.handleChange((e.target as HTMLInputElement).value)"
            />
          </template>
        </form.Field>
      </div>
      <button type="submit">提交</button>
    </form>
  </div>
</template>
```

从这里开始，您就可以探索 TanStack Form 的所有其他功能了！
