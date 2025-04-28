---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-12T04:08:52.717Z'
id: overview
title: 概述
---

TanStack Form 是处理 Web 应用中表单的终极解决方案，提供了一种强大而灵活的表单管理方法。它具备一流的 TypeScript 支持、无头 UI 组件 (headless UI components) 和框架无关的设计 (framework-agnostic design)，简化了表单处理流程，并确保在不同前端框架中都能获得无缝体验。

## 动机

大多数 Web 框架并未提供全面的表单处理方案，开发者不得不自行实现定制方案或依赖功能有限的库。这通常会导致一致性缺失、性能低下以及开发时间增加。TanStack Form 旨在通过提供一套强大易用的全功能表单管理方案来解决这些问题。

使用 TanStack Form，开发者可以应对以下常见的表单相关挑战：

- 响应式数据绑定 (Reactive data binding) 和状态管理 (state management)
- 复杂验证 (validation) 和错误处理 (error handling)
- 无障碍访问 (accessibility) 和响应式设计 (responsive design)
- 国际化 (internationalization) 和本地化 (localization)
- 跨平台兼容性 (cross-platform compatibility) 和自定义样式 (custom styling)

通过为这些挑战提供完整解决方案，TanStack Form 让开发者能够轻松构建健壮且用户友好的表单。

## 少说多做，直接看代码！

下面的示例展示了 TanStack Form 与 React 框架适配器配合使用的实际效果：

[在 CodeSandbox 中打开](https://codesandbox.io/s/github/tanstack/form/tree/main/examples/react/simple)

```tsx
import * as React from 'react'
import { createRoot } from 'react-dom/client'
import { useForm } from '@tanstack/react-form'
import type { AnyFieldApi } from '@tanstack/react-form'

function FieldInfo({ field }: { field: AnyFieldApi }) {
  return (
    <>
      {field.state.meta.isTouched && field.state.meta.errors.length ? (
        <em>{field.state.meta.errors.join(', ')}</em>
      ) : null}
      {field.state.meta.isValidating ? 'Validating...' : null}
    </>
  )
}

export default function App() {
  const form = useForm({
    defaultValues: {
      firstName: '',
      lastName: '',
    },
    onSubmit: async ({ value }) => {
      // Do something with form data
      console.log(value)
    },
  })

  return (
    <div>
      <h1>Simple Form Example</h1>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          form.handleSubmit()
        }}
      >
        <div>
          {/* A type-safe field component*/}
          <form.Field
            name="firstName"
            validators={{
              onChange: ({ value }) =>
                !value
                  ? 'A first name is required'
                  : value.length < 3
                    ? 'First name must be at least 3 characters'
                    : undefined,
              onChangeAsyncDebounceMs: 500,
              onChangeAsync: async ({ value }) => {
                await new Promise((resolve) => setTimeout(resolve, 1000))
                return (
                  value.includes('error') && 'No "error" allowed in first name'
                )
              },
            }}
            children={(field) => {
              // Avoid hasty abstractions. Render props are great!
              return (
                <>
                  <label htmlFor={field.name}>First Name:</label>
                  <input
                    id={field.name}
                    name={field.name}
                    value={field.state.value}
                    onBlur={field.handleBlur}
                    onChange={(e) => field.handleChange(e.target.value)}
                  />
                  <FieldInfo field={field} />
                </>
              )
            }}
          />
        </div>
        <div>
          <form.Field
            name="lastName"
            children={(field) => (
              <>
                <label htmlFor={field.name}>Last Name:</label>
                <input
                  id={field.name}
                  name={field.name}
                  value={field.state.value}
                  onBlur={field.handleBlur}
                  onChange={(e) => field.handleChange(e.target.value)}
                />
                <FieldInfo field={field} />
              </>
            )}
          />
        </div>
        <form.Subscribe
          selector={(state) => [state.canSubmit, state.isSubmitting]}
          children={([canSubmit, isSubmitting]) => (
            <button type="submit" disabled={!canSubmit}>
              {isSubmitting ? '...' : 'Submit'}
            </button>
          )}
        />
      </form>
    </div>
  )
}

const rootElement = document.getElementById('root')!

createRoot(rootElement).render(<App />)
```

## 既然被说服了，接下来该怎么做？

- 通过我们的详细[入门指南](../installation)和[API 参考文档](../reference/classes/formapi)按自己的节奏学习 TanStack Form
