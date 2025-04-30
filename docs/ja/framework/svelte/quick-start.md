---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:24:35.000Z'
id: quick-start
title: クイックスタート
---

# クイックスタート

TanStack Formを始めるための最低限の手順は、フォームを作成してフィールドを追加することです。この例にはまだバリデーションやエラーハンドリングは含まれていないことに注意してください。

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      fullName: '',
    },
    onSubmit: async ({ value }) => {
      // フォームデータを処理
      console.log(value)
    },
  }))
</script>

<div>
  <h1>シンプルなフォーム例</h1>
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
    <button type="submit">送信</button>
  </form>
</div>
```

ここから、TanStack Formの他のすべての機能を探索する準備が整いました！
