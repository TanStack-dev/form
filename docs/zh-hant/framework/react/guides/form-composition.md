---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-25T20:36:29.116Z'
id: form-composition
title: 表單組合
---
# 表單組合 (Form Composition)

對 TanStack Form 的一個常見批評是其開箱即用的冗長性。雖然這對於教育目的可能有用 — 幫助強化對 API 的理解 — 但在生產環境中使用並不理想。

因此，雖然 `form.Field` 提供了 TanStack Form 最強大且靈活的使用方式，但我們也提供了封裝它的 API，使你的應用程式代碼更加簡潔。

## 動機

大多數網頁框架並未提供全面的表單處理解決方案，迫使開發者自行創建自訂實現或依賴功能較弱的函式庫。這通常導致缺乏一致性、效能不佳和開發時間增加。TanStack Form 旨在通過提供一個強大且易於使用的全方位表單管理解決方案來應對這些挑戰。

## 自訂表單鉤子 (Custom Form Hooks)

最強大的表單組合方式是創建自訂表單鉤子。這允許你創建一個符合應用程式需求的表單鉤子，包括預先綁定的自訂 UI 元件等。

最基本的 `createFormHook` 是一個接收 `fieldContext` 和 `formContext` 並返回 `useAppForm` 鉤子的函式。

> 這個未自訂的 `useAppForm` 鉤子與 `useForm` 相同，但隨著我們為 `createFormHook` 添加更多選項，這將很快改變。

```tsx
import { createFormHookContexts, createFormHook } from '@tanstack/react-form'

// 導出 useFieldContext 供自訂元件使用
export const { fieldContext, formContext, useFieldContext } =
  createFormHookContexts()

const { useAppForm } = createFormHook({
  fieldContext,
  formContext,
  // 我們稍後會詳細介紹這些選項
  fieldComponents: {},
  formComponents: {},
})

function App() {
  const form = useAppForm({
    // 支援所有 useForm 選項
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return <form.Field /> // ...
}
```

### 預綁定欄位元件 (Pre-bound Field Components)

一旦這個基礎架構就位，你就可以開始向表單鉤子添加自訂欄位和表單元件。

> 注意：`useFieldContext` 必須與自訂表單上下文中導出的相同

```tsx
import { useFieldContext } from './form-context.tsx'

export function TextField({ label }: { label: string }) {
  // `Field` 推斷其 `value` 類型應為 `string`
  const field = useFieldContext<string>()
  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}
```

然後你可以將此元件註冊到表單鉤子中。

```tsx
import { TextField } from './text-field.tsx'

const { useAppForm } = createFormHook({
  fieldContext,
  formContext,
  fieldComponents: {
    TextField,
  },
  formComponents: {},
})
```

並在表單中使用它：

```tsx
function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return (
    // 注意使用 `AppField` 而非 `Field`；`AppField` 提供了所需的上下文
    <form.AppField
      name="firstName"
      children={(field) => <field.TextField label="First Name" />}
    />
  )
}
```

這不僅允許你重用共享元件的 UI，還保留了 TanStack Form 所期望的類型安全性：如果拼錯 `name`，將會收到 TypeScript 錯誤。

#### 關於效能的說明

雖然上下文 (context) 是 React 生態系統中的一個有價值的工具，但許多使用者合理擔心通過上下文提供的響應式值會導致不必要的重新渲染。

> 不熟悉這個效能問題？[Mark Erikson 解釋 Redux 如何解決這些問題的部落格文章](https://blog.isquaredsoftware.com/2021/01/context-redux-differences/)是一個很好的起點。

雖然這是一個值得提出的問題，但對 TanStack Form 來說不是問題；通過上下文提供的值本身不是響應式的，而是具有響應式屬性的靜態類實例（使用 TanStack Store 作為我們的信號實現來驅動這一切）。

### 預綁定表單元件 (Pre-bound Form Components)

雖然 `form.AppField` 解決了許多關於欄位樣板代碼和可重用性的問題，但它並未解決表單樣板代碼和可重用性的問題。

特別是，能夠共享 `form.Subscribe` 的實例，例如用於響應式表單提交按鈕，是一個常見的使用案例。

```tsx
function SubscribeButton({ label }: { label: string }) {
  const form = useFormContext()
  return (
    <form.Subscribe selector={(state) => state.isSubmitting}>
      {(isSubmitting) => <button disabled={isSubmitting}>{label}</button>}
    </form.Subscribe>
  )
}

const { useAppForm, withForm } = createFormHook({
  fieldComponents: {},
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return (
    <form.AppForm>
      // 注意 `AppForm` 元件包裝器；`AppForm` 提供了所需的上下文
      <form.SubscribeButton label="Submit" />
    </form.AppForm>
  )
}
```

## 將大型表單拆分為小部分

有時表單會變得非常大；這只是有時會發生的情況。雖然 TanStack Form 能很好地支援大型表單，但處理數百或數千行程式碼的檔案從來都不是一件有趣的事。

為了解決這個問題，我們支援使用 `withForm` 高階元件 (HOC) 將表單拆分為較小的部分。

```tsx
const { useAppForm, withForm } = createFormHook({
  fieldComponents: {
    TextField,
  },
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

const ChildForm = withForm({
  // 這些值僅用於類型檢查，不在運行時使用
  // 這允許你從 `formOptions` 中 `...formOpts` 而無需重新聲明選項
  defaultValues: {
    firstName: 'John',
    lastName: 'Doe',
  },
  // 可選，但除了 `form` 外還為 `render` 函式添加屬性
  props: {
    // 這些屬性也設置為 `render` 函式的默認值
    title: 'Child Form',
  },
  render: function Render({ form, title }) {
    return (
      <div>
        <p>{title}</p>
        <form.AppField
          name="firstName"
          children={(field) => <field.TextField label="First Name" />}
        />
        <form.AppForm>
          <form.SubscribeButton label="Submit" />
        </form.AppForm>
      </div>
    )
  },
})

function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return <ChildForm form={form} title={'Testing'} />
}
```

### `withForm` 常見問題

> 為什麼使用高階元件而非鉤子？

雖然鉤子是 React 的未來，但高階元件仍然是組合的強大工具。特別是，`useForm` 的 API 使我們能夠擁有強大的類型安全性，而無需使用者傳遞泛型。

> 為什麼我在 `render` 中收到關於鉤子的 ESLint 錯誤？

ESLint 在函式的頂層尋找鉤子，而 `render` 可能不會被識別為頂層元件，具體取決於你如何定義它。

```tsx
// 這將導致使用鉤子時出現 ESLint 錯誤
const ChildForm = withForm({
  // ...
  render: ({ form, title }) => {
    // ...
  },
})
```

```tsx
// 這樣可以正常工作
const ChildForm = withForm({
  // ...
  render: function Render({ form, title }) {
    // ...
  },
})
```

## 樹搖晃表單和欄位元件 (Tree-shaking Form and Field Components)

雖然上述範例對於入門非常有用，但它們並不適合某些使用案例，例如你可能擁有數百個表單和欄位元件的情況。
特別是，你可能不希望在所有使用表單鉤子的檔案的套件中包含所有的表單和欄位元件。

為了解決這個問題，你可以將 `createFormHook` TanStack API 與 React 的 `lazy` 和 `Suspense` 元件結合使用：

```typescript
// src/hooks/form-context.ts
import { createFormHookContexts } from '@tanstack/react-form'

export const { fieldContext, useFieldContext, formContext, useFormContext } =
  createFormHookContexts()
```

```tsx
// src/components/text-field.tsx
import { useFieldContext } from '../hooks/form-context.tsx'

export default function TextField({ label }: { label: string }) {
  const field = useFieldContext<string>()

  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}
```

```tsx
// src/hooks/form.ts
import { lazy } from 'react'
import { createFormHook } from '@tanstack/react-form'

const TextField = lazy(() => import('../components/text-fields.tsx'))

const { useAppForm, withForm } = createFormHook({
  fieldContext,
  formContext,
  fieldComponents: {
    TextField,
  },
  formComponents: {},
})
```

```tsx
// src/App.tsx
import { Suspense } from 'react'
import { PeoplePage } from './features/people/page.tsx'

export default function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <PeopleForm />
    </Suspense>
  )
}
```

這將在載入 `TextField` 元件時顯示 Suspense 回退內容，然後在載入完成後渲染表單。

## 綜合應用

現在我們已經介紹了創建自訂表單鉤子的基礎知識，讓我們將所有內容整合到一個範例中。

```tsx
// /src/hooks/form.ts，用於整個應用程式
const { fieldContext, useFieldContext, formContext, useFormContext } =
  createFormHookContexts()

function TextField({ label }: { label: string }) {
  const field = useFieldContext<string>()
  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}

function SubscribeButton({ label }: { label: string }) {
  const form = useFormContext()
  return (
    <form.Subscribe selector={(state) => state.isSubmitting}>
      {(isSubmitting) => <button disabled={isSubmitting}>{label}</button>}
    </form.Subscribe>
  )
}

const { useAppForm, withForm } = createFormHook({
  fieldComponents: {
    TextField,
  },
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

// /src/features/people/shared-form.ts，用於 `people` 功能
const formOpts = formOptions({
  defaultValues: {
    firstName: 'John',
    lastName: 'Doe',
  },
})

// /src/features/people/nested-form.ts，用於 `people` 頁面
const ChildForm = withForm({
  ...formOpts,
  // 可選，但為 `render` 函式添加 `form` 之外的屬性
  props: {
    title: 'Child Form',
  },
  render: ({ form, title }) => {
    return (
      <div>
        <p>{title}</p>
        <form.AppField
          name="firstName"
          children={(field) => <field.TextField label="First Name" />}
        />
        <form.AppForm>
          <form.SubscribeButton label="Submit" />
        </form.AppForm>
      </div>
    )
  },
})

// /src/features/people/page.ts
const Parent = () => {
  const form = useAppForm({
    ...formOpts,
  })

  return <ChildForm form={form} title={'Testing'} />
}
```

## API 使用指南

以下圖表可幫助你決定應該使用哪些 API：

![](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/react_form_composability.svg)
