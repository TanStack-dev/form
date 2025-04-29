---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-29T23:14:15.161Z'
id: quick-start
title: 快速开始
---

## 快速入门

使用 TanStack Form 的最基础操作是创建一个表单并添加字段。请注意，当前示例尚未包含任何验证或错误处理功能。

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      fullName: '',
    },
    onSubmit: async ({ value }) => {
      // 对表单数据进行操作
      console.log(value)
    },
  }))
</script>

<div>
  <h1>简单表单示例</h1>
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

从这里开始，您就可以探索 TanStack Form 的所有其他功能了！
