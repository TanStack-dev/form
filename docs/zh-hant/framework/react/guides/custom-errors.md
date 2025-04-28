---
source-updated-at: '2025-03-04T14:23:57.000Z'
translation-updated-at: '2025-04-25T20:34:22.749Z'
id: custom-errors
title: 自訂錯誤
---

TanStack Form 提供了完整的靈活性，讓您可以從驗證器返回任何類型的錯誤值。字串錯誤是最常見且易於處理的，但該函式庫允許您從驗證器返回任何類型的值。

一般情況下，任何真值 (truthy value) 都會被視為錯誤，並將表單或欄位標記為無效；而假值 (falsy value)（`false`、`undefined`、`null` 等）則表示沒有錯誤，表單或欄位是有效的。

## 從表單返回字串值

```tsx
<form.Field
  name="username"
  validators={{
    onChange: ({ value }) =>
      value.length < 3 ? 'Username must be at least 3 characters' : undefined,
  }}
/>
```

對於影響多個欄位的表單級驗證：

```tsx
const form = useForm({
  defaultValues: {
    username: '',
    email: '',
  },
  validators: {
    onChange: ({ value }) => {
      return {
        fields: {
          username:
            value.username.length < 3 ? 'Username too short' : undefined,
          email: !value.email.includes('@') ? 'Invalid email' : undefined,
        },
      }
    },
  },
})
```

字串錯誤是最常見的類型，並且可以輕鬆地在您的 UI 中顯示：

```tsx
{
  field.state.meta.errors.map((error, i) => (
    <div key={i} className="error">
      {error}
    </div>
  ))
}
```

### 數字

用於表示數量、閾值或大小：

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 18 ? 18 - value : undefined),
  }}
/>
```

在 UI 中顯示：

```tsx
{
  /* TypeScript 會根據您的驗證器知道錯誤是一個數字 */
}
;<div className="error">
  You need {field.state.meta.errors[0]} more years to be eligible
</div>
```

### 布林值

簡單的標誌來指示錯誤狀態：

```tsx
<form.Field
  name="accepted"
  validators={{
    onChange: ({ value }) => (!value ? true : undefined),
  }}
/>
```

在 UI 中顯示：

```tsx
{
  field.state.meta.errors[0] === true && (
    <div className="error">You must accept the terms</div>
  )
}
```

### 物件

具有多個屬性的豐富錯誤物件：

```tsx
<form.Field
  name="email"
  validators={{
    onChange: ({ value }) => {
      if (!value.includes('@')) {
        return {
          message: 'Invalid email format',
          severity: 'error',
          code: 1001,
        }
      }
      return undefined
    },
  }}
/>
```

在 UI 中顯示：

```tsx
{
  typeof field.state.meta.errors[0] === 'object' && (
    <div className={`error ${field.state.meta.errors[0].severity}`}>
      {field.state.meta.errors[0].message}
      <small> (Code: {field.state.meta.errors[0].code})</small>
    </div>
  )
}
```

在上面的範例中，它取決於您想要顯示的事件錯誤。

### 陣列

單個欄位的多個錯誤訊息：

```tsx
<form.Field
  name="password"
  validators={{
    onChange: ({ value }) => {
      const errors = []
      if (value.length < 8) errors.push('Password too short')
      if (!/[A-Z]/.test(value)) errors.push('Missing uppercase letter')
      if (!/[0-9]/.test(value)) errors.push('Missing number')

      return errors.length ? errors : undefined
    },
  }}
/>
```

在 UI 中顯示：

```tsx
{
  Array.isArray(field.state.meta.errors) && (
    <ul className="error-list">
      {field.state.meta.errors.map((err, i) => (
        <li key={i}>{err}</li>
      ))}
    </ul>
  )
}
```

## 欄位上的 `disableErrorFlat` 屬性

預設情況下，TanStack Form 會將所有驗證來源（onChange、onBlur、onSubmit）的錯誤扁平化為單一的 `errors` 陣列。`disableErrorFlat` 屬性可以保留錯誤來源：

```tsx
<form.Field
  name="email"
  disableErrorFlat
  validators={{
    onChange: ({ value }) =>
      !value.includes('@') ? 'Invalid email format' : undefined,
    onBlur: ({ value }) =>
      !value.endsWith('.com') ? 'Only .com domains allowed' : undefined,
    onSubmit: ({ value }) => (value.length < 5 ? 'Email too short' : undefined),
  }}
/>
```

如果沒有 `disableErrorFlat`，所有錯誤將被合併到 `field.state.meta.errors` 中。有了它，您可以按來源訪問錯誤：

```tsx
{
  field.state.meta.errorMap.onChange && (
    <div className="real-time-error">{field.state.meta.errorMap.onChange}</div>
  )
}

{
  field.state.meta.errorMap.onBlur && (
    <div className="blur-feedback">{field.state.meta.errorMap.onBlur}</div>
  )
}

{
  field.state.meta.errorMap.onSubmit && (
    <div className="submit-error">{field.state.meta.errorMap.onSubmit}</div>
  )
}
```

這對於以下情況非常有用：

- 以不同的 UI 樣式顯示不同類型的錯誤
- 優先處理錯誤（例如，更突出地顯示提交錯誤）
- 實現錯誤的漸進式揭露

## `errors` 和 `errorMap` 的類型安全

TanStack Form 為錯誤處理提供了強大的類型安全。`errorMap` 中的每個鍵都具有其對應驗證器返回的確切類型，而 `errors` 陣列則包含來自所有驗證器的所有可能錯誤值的聯合類型：

```tsx
<form.Field
  name="password"
  validators={{
    onChange: ({ value }) => {
      // 這會返回一個字串或 undefined
      return value.length < 8 ? 'Too short' : undefined
    },
    onBlur: ({ value }) => {
      // 這會返回一個物件或 undefined
      if (!/[A-Z]/.test(value)) {
        return { message: 'Missing uppercase', level: 'warning' }
      }
      return undefined
    },
  }}
  children={(field) => {
    // TypeScript 知道 errors[0] 可以是 string | { message: string, level: string } | undefined
    const error = field.state.meta.errors[0]

    // 類型安全的錯誤處理
    if (typeof error === 'string') {
      return <div className="string-error">{error}</div>
    } else if (error && typeof error === 'object') {
      return <div className={error.level}>{error.message}</div>
    }

    return null
  }}
/>
```

`errorMap` 屬性也是完全類型化的，與您的驗證函式的返回類型匹配：

```tsx
// 使用 disableErrorFlat
<form.Field
  name="email"
  disableErrorFlat
  validators={{
    onChange: ({ value }): string | undefined =>
      !value.includes("@") ? "Invalid email" : undefined,
    onBlur: ({ value }): { code: number, message: string } | undefined =>
      !value.endsWith(".com") ? { code: 100, message: "Wrong domain" } : undefined
  }}
  children={(field) => {
    // TypeScript 知道每個錯誤來源的確切類型
    const onChangeError: string | undefined = field.state.meta.errorMap.onChange;
    const onBlurError: { code: number, message: string } | undefined = field.state.meta.errorMap.onBlur;

    return (/* ... */);
  }}
/>
```

這種類型安全有助於在編譯時而不是運行時捕獲錯誤，使您的代碼更加可靠和易於維護。
