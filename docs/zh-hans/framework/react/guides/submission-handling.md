---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-12T04:09:04.203Z'
id: submission-handling
title: 提交处理
---

在需要处理多种表单提交类型的场景下（例如，一个表单同时包含导航至子表单的按钮和执行标准提交的按钮），可以利用 `onSubmitMeta` 属性和 `handleSubmit` 函数的重载机制。

## 基础用法

首先需要定义 `form.onSubmitMeta` 属性的默认状态：

```tsx
const form = useForm({
  defaultValues: {
    firstName: 'Rick',
  },
  // {} 是传递给 `onSubmit` 的 `meta` 属性的默认值
  onSubmitMeta: {} as { lastName: string },
  onSubmit: async ({ value, meta }) => {
    // 处理通过 handleSubmit 传递的值
    console.log(`${value.firstName} - ${meta}`)
  },
})
```

注意：`onSubmitMeta` 的默认状态为 `never` 类型，因此若未提供该属性却尝试在 `handleSubmit` 或 `onSubmit` 中访问它，将会报错。

调用 `onSubmit` 时，可按如下方式传入预定义的元数据：

```tsx
<form
  onSubmit={(e) => {
    e.preventDefault()
    e.stopPropagation()
    form.handleSubmit({
      lastName: 'Astley',
    })
  }}
></form>
```
