---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-29T23:11:34.854Z'
id: quick-start
title: 快速開始
---

## 快速開始

使用 TanStack Form 的基礎入門步驟是建立一個表單並加入欄位。請注意，此範例尚未包含任何驗證或錯誤處理功能。

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      fullName: '',
    },
    onSubmit: async ({ value }) => {
      // 對表單資料進行操作
      console.log(value)
    },
  }))
</script>

<div>
  <h1>簡單表單範例</h1>
  <form
    onsubmit={(e) => {
      e.preventDefault()
      e.stopPropagation()
      form.handleSubmit()
    }}
  >
    <div>
      <form.Field name="fullName">
        {#snippet children(field)}
          <input
            name={field.name}
            value={field.state.value}
            onblur={field.handleBlur}
            oninput={(e) => field.handleChange(e.target.value)}
          />
        {/snippet}
      </form.Field>
    </div>
    <button type="submit">提交</button>
  </form>
</div>
```

從這裡開始，您就可以準備探索 TanStack Form 的所有其他功能了！
