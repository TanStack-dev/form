---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:17:53.947Z'
id: submission-handling
title: Submission handling
---

## 傳遞額外資料至提交處理

您可能需要處理多種提交行為類型，例如返回其他頁面或停留在表單上。您可以透過指定 `onSubmitMeta` 屬性來實現此功能，這些元資料將會傳遞至 `onSubmit` 函式。

> 注意：若呼叫 `form.handleSubmit()` 時未提供元資料，則會使用預設值。

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null
}

// 呼叫 form.handleSubmit() 時不需強制提供元資料。
// 指定未傳遞元資料時使用的預設值
const defaultMeta: FormMeta = {
  submitAction: null,
}

const form = useForm({
  defaultValues: {
    data: '',
  },
  // 定義提交時預期的元資料值
  onSubmitMeta: defaultMeta,
  onSubmit: async ({ value, meta }) => {
    // 處理透過 handleSubmit 傳遞的值
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
          // 覆寫 onSubmitMeta 中指定的預設值
          form.handleSubmit({ submitAction: 'continue' })
        }
      "
    >
      提交並繼續
    </button>
    <button
      type="submit"
      @click="() => form.handleSubmit({ submitAction: 'backToMenu' })"
    >
      提交並返回選單
    </button>
  </form>
</template>
```

## 使用標準結構描述轉換資料

雖然 Tanstack Form 提供了[標準結構描述支援 (Standard Schema support)](./validation.md) 進行驗證，但不會保留結構描述的輸出資料。

傳遞至 `onSubmit` 函式的值永遠是輸入資料。若要取得標準結構描述的輸出資料，請在 `onSubmit` 函式中解析它：

```vue
<script setup lang="ts">
import { z } from 'zod'
import { useForm } from '@tanstack/vue-form'

const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack form 使用標準結構描述的輸入類型
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
    // 透過結構描述取得轉換後的值
    const result = schema.parse(value)
    const outputAge: number = result.age
  },
})
</script>
<template>
  <!-- ... -->
</template>
```
