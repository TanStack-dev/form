---
source-updated-at: '2025-03-31T17:05:18.000Z'
translation-updated-at: '2025-04-25T20:31:15.746Z'
id: quick-start
title: 快速開始
---
## 快速開始

使用 TanStack Form 最基本的入門步驟是建立一個表單並加入欄位。請注意，此範例尚未包含任何驗證或錯誤處理功能。

```vue
<!-- App.vue -->
<script setup>
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    fullName: '',
  },
  onSubmit: async ({ value }) => {
    // 在此處理表單資料
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

從這裡開始，您就可以準備探索 TanStack Form 的所有其他功能了！
