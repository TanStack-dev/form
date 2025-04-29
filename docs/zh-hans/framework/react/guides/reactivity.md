---
source-updated-at: '2025-03-20T03:01:05.000Z'
translation-updated-at: '2025-04-12T04:08:53.631Z'
id: reactivity
title: 响应式
---

Tanstack Form 在与表单交互时不会触发重新渲染。因此您可能会发现直接使用表单或字段状态值无法获得预期效果。

如需访问响应式值，您需要通过以下两种方法之一进行订阅：`useStore` 钩子或 `form.Subscribe` 组件。

这些订阅的典型使用场景包括：渲染实时更新的字段值、根据条件决定渲染内容，或在组件逻辑中使用字段值。

> 若需对特定触发器作出"响应"，请查阅 [监听器](./listeners.md) API。

## useStore

当您需要在组件逻辑中访问表单值时，`useStore` 钩子是最佳选择。该钩子接收两个参数：第一个是表单存储 (form store)，第二个是用于精确选择订阅内容的选择器 (selector)。

```tsx
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
```

您可以通过选择器访问表单状态的任何部分。

> 注意：当订阅的值发生变化时，`useStore` 会导致整个组件重新渲染。

虽然技术上可以省略选择器，但请避免这种操作，因为省略后将导致表单状态任何变化都会触发不必要的重新渲染。

## form.Subscribe

`form.Subscribe` 组件最适合在需要根据表单状态更新 UI 时使用。例如：根据表单字段值显示或隐藏某些界面元素。

```tsx
<form.Subscribe
  selector={(state) => state.values.firstName}
  children={(firstName) => (
    <form.Field>
      {(field) => (
        <input
          name="lastName"
          value={field.state.lastName}
          onChange={field.handleChange}
        />
      )}
    </form.Field>
  )}
/>
```

> `form.Subscribe` 组件不会触发组件级重新渲染。当订阅的值变化时，仅该组件本身会重新渲染。

选择使用 `useStore` 还是 `form.Subscribe` 主要取决于使用场景：若涉及 UI 渲染，优先选用具有优化特性的 `form.Subscribe`；若需要在逻辑中实现响应式，则应当选择 `useStore`。
