---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-29T23:28:37.258Z'
id: quick-start
title: 快速開始
---

TanStack Form 與您以往使用過的大多數表單函式庫不同。它專為大規模生產環境設計，強調類型安全、效能和組合性，提供無與倫比的開發體驗。

因此，我們圍繞函式庫的使用[制定了一套哲學](/form/latest/docs/philosophy)，重視可擴展性和長期開發體驗，而非簡短可分享的程式碼片段。

以下是一個遵循我們許多最佳實踐的表單範例，讓您在短暫上手後就能快速開發高複雜度的表單：

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { createFormHook, createFormHookContexts } from '@tanstack/react-form'
// 預先綁定表單鉤子事件的表單元件；詳見我們的「表單組合」指南
import { TextField, NumberField, SubmitButton } from '~our-app/ui-library'
// 我們也支援 Valibot、ArkType 和其他標準的 schema 函式庫
import { z } from 'zod'

const { fieldContext, formContext } = createFormHookContexts()

// 讓我們能將元件綁定到表單上，保持類型安全同時減少生產環境的樣板程式碼
// 定義一次即可在整個應用中生成一致的表單實例
const { useAppForm } = createFormHook({
  fieldComponents: {
    TextField,
    NumberField,
  },
  formComponents: {
    SubmitButton,
  },
  fieldContext,
  formContext,
})

const PeoplePage = () => {
  const form = useAppForm({
    defaultValues: {
      username: '',
      age: 0,
    },
    validators: {
      // 傳入 schema 或函數進行驗證
      onChange: z.object({
        username: z.string(),
        age: z.number().min(13),
      }),
    },
    onSubmit: ({ value }) => {
      // 處理表單資料
      alert(JSON.stringify(value, null, 2))
    },
  })

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault()
        form.handleSubmit()
      }}
    >
      <h1>Personal Information</h1>
      {/* 元件綁定到 `form` 和 `field` 以確保極致的類型安全 */}
      {/* 使用 `form.AppField` 渲染綁定到單一欄位的元件 */}
      <form.AppField
        name="username"
        children={(field) => <field.TextField label="Full Name" />}
      />
      {/* 若拼寫錯誤，"name" 屬性會拋出 TypeScript 錯誤 */}
      <form.AppField
        name="age"
        children={(field) => <field.NumberField label="Age" />}
      />
      {/* `form.AppForm` 中的元件可存取表單上下文 */}
      <form.AppForm>
        <form.SubmitButton />
      </form.AppForm>
    </form>
  )
}

const rootElement = document.getElementById('root')!
ReactDOM.createRoot(rootElement).render(<PeoplePage />)
```

雖然我們通常建議使用 `createFormHook` 以長期減少樣板程式碼，但我們也支援使用 `useForm` 和 `form.Field` 來實現一次性元件和其他行為：

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { useForm } from '@tanstack/react-form'

const PeoplePage = () => {
  const form = useForm({
    defaultValues: {
      username: '',
      age: 0,
    },
    onSubmit: ({ value }) => {
      // 處理表單資料
      alert(JSON.stringify(value, null, 2))
    },
  })

  return (
    <form.Field
      name="age"
      validators={{
        // 可選擇表單範圍或欄位特定的驗證器
        onChange: ({ value }) =>
          value > 13 ? undefined : 'Must be 13 or older',
      }}
      children={(field) => (
        <>
          <input
            name={field.name}
            value={field.state.value}
            onBlur={field.handleBlur}
            type="number"
            onChange={(e) => field.handleChange(e.target.valueAsNumber)}
          />
          {!field.state.meta.isValid && (
            <em>{field.state.meta.errors.join(',')}</em>
          )}
        </>
      )}
    />
  )
}

const rootElement = document.getElementById('root')!
ReactDOM.createRoot(rootElement).render(<PeoplePage />)
```

`useForm` 的所有屬性都可用於 `useAppForm`，而 `form.Field` 的所有屬性也都可用於 `form.AppField`。
