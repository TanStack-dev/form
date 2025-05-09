---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-29T23:28:35.764Z'
id: overview
title: 概述
---

TanStack Form 是處理網頁應用程式中表單的終極解決方案，提供強大且靈活的表單管理方法。它具備一流的 TypeScript 支援、無頭 UI 元件 (headless UI components) 以及框架無關的設計 (framework-agnostic design)，能簡化表單處理流程，並確保在不同前端框架 (front-end frameworks) 間獲得無縫體驗。

## 動機

大多數網頁框架 (web frameworks) 並未提供完整的表單處理解決方案，迫使開發者必須自行實作客製化方案或依賴功能較弱的函式庫。這通常會導致一致性不足、效能低落，以及開發時間增加等問題。TanStack Form 旨在透過提供一個強大且易用的全方位表單管理方案來解決這些挑戰。

使用 TanStack Form，開發者能夠應對以下常見的表單相關挑戰：

- 響應式資料綁定 (Reactive data binding) 與狀態管理 (state management)
- 複雜驗證 (validation) 與錯誤處理 (error handling)
- 無障礙性 (accessibility) 與響應式設計 (responsive design)
- 國際化 (internationalization) 與本地化 (localization)
- 跨平台相容性 (cross-platform compatibility) 與自訂樣式 (custom styling)

透過為這些挑戰提供完整解決方案，TanStack Form 讓開發者能輕鬆建構穩健且使用者友善的表單。

## 說夠了，直接看程式碼吧！

以下範例展示 TanStack Form 與 React 框架轉接器 (framework adapter) 的實際應用：

[在 CodeSandbox 中開啟](https://codesandbox.io/s/github/tanstack/form/tree/main/examples/react/simple)

```tsx
import * as React from 'react'
import { createRoot } from 'react-dom/client'
import { useForm } from '@tanstack/react-form'
import type { AnyFieldApi } from '@tanstack/react-form'

function FieldInfo({ field }: { field: AnyFieldApi }) {
  return (
    <>
      {field.state.meta.isTouched && !field.state.meta.isValid ? (
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

## 你成功說服我了，接下來呢？

- 透過我們詳盡的[入門指南](../installation)與[API 參考文件](../reference/classes/formapi)，按照自己的步調學習 TanStack Form
