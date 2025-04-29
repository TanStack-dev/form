---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-25T20:31:19.846Z'
id: quick-start
title: 快速開始
---

## 快速開始

使用 TanStack Form 最基本的入門步驟是建立一個表單並添加欄位。請注意，此範例尚未包含任何驗證或錯誤處理功能。

```tsx
import { createForm } from '@tanstack/solid-form'

function App() {
  const form = createForm(() => ({
    defaultValues: {
      fullName: '',
    },
    onSubmit: async ({ value }) => {
      // 對表單資料進行處理
      console.log(value)
    },
  }))

  return (
    <div>
      <h1>簡單表單範例</h1>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          form.handleSubmit()
        }}
      >
        <div>
          <form.Field
            name="fullName"
            children={(field) => (
              <input
                name={field().name}
                value={field().state.value}
                onBlur={field().handleBlur}
                onInput={(e) => field().handleChange(e.target.value)}
              />
            )}
          />
        </div>
        <button type="submit">提交</button>
      </form>
    </div>
  )
}
```

從這裡開始，您就可以準備探索 TanStack Form 的所有其他功能了！
