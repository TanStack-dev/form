---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-12T04:08:24.395Z'
id: quick-start
title: 快速开始
---
## 快速入门

开始使用 TanStack Form 的最低要求是创建一个表单并添加一个字段。请注意，这个示例尚未包含任何验证或错误处理功能。

```tsx
import { createForm } from '@tanstack/solid-form'

function App() {
  const form = createForm(() => ({
    defaultValues: {
      fullName: '',
    },
    onSubmit: async ({ value }) => {
      // 对表单数据进行处理
      console.log(value)
    },
  }))

  return (
    <div>
      <h1>简单表单示例</h1>
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

从这里开始，您就可以探索 TanStack Form 的所有其他功能了！
