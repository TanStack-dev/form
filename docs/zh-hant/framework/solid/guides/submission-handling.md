---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:18:02.604Z'
id: submission-handling
title: Submission handling
---

## 傳遞額外資料至提交處理

您可能有多種提交行為類型，例如返回其他頁面或停留在表單上。您可以通過指定 `onSubmitMeta` 屬性來實現這一點。這些元資料將會傳遞至 `onSubmit` 函數。

> 注意：若 `form.handleSubmit()` 在沒有元資料的情況下被呼叫，將會使用預設提供的值。

```tsx
import { createForm } from '@tanstack/solid-form'

type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null
}

// 呼叫 form.handleSubmit() 時不強制要求傳遞元資料。
// 指定當未傳遞元資料時應使用的預設值
const defaultMeta: FormMeta = {
  submitAction: null,
}

export default function App() {
  const form = createForm(() => ({
    defaultValues: {
      data: '',
    },
    // 定義提交時預期的元資料值
    onSubmitMeta: defaultMeta,
    onSubmit: async ({ value, meta }) => {
      // 使用透過 handleSubmit 傳遞的值進行操作
      console.log(`Selected action - ${meta.submitAction}`, value)
    },
  }))

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault()
        e.stopPropagation()
      }}
    >
      {/* ... */}
      <button
        type="submit"
        // 覆寫 onSubmitMeta 中指定的預設值
        onClick={() => form.handleSubmit({ submitAction: 'continue' })}
      >
        提交並繼續
      </button>
      <button
        type="submit"
        onClick={() => form.handleSubmit({ submitAction: 'backToMenu' })}
      >
        提交並返回選單
      </button>
    </form>
  )
}
```

## 使用標準結構描述 (Standard Schemas) 轉換資料

雖然 Tanstack Form 提供了[標準結構描述支援](./validation.md)來進行驗證，但它不會保留結構描述的輸出資料。

傳遞至 `onSubmit` 函數的值始終會是輸入資料。若要取得標準結構描述的輸出資料，請在 `onSubmit` 函數中解析它：

```tsx
import { createForm } from '@tanstack/solid-form'
import { z } from 'zod'

// ...

const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Form 使用標準結構描述的輸入類型
const defaultValues: z.input<typeof schema> = {
  age: '13',
}

const form = createForm(() => ({
  defaultValues,
  validators: {
    onChange: schema,
  },
  onSubmit: ({ value }) => {
    const inputAge: string = value.age
    // 透過結構描述解析以取得轉換後的值
    const result = schema.parse(value)
    const outputAge: number = result.age
  },
}))
```
