---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-29T23:30:22.189Z'
id: form-validation
title: 表單驗證
---

## 表單與欄位驗證 (Form and Field Validation)

TanStack Form 的核心功能之一是驗證機制，它讓驗證過程高度可自訂：

- 您可以控制驗證觸發時機（例如：變更時、輸入時、失焦時、提交時...）
- 驗證規則可以在欄位層級或表單層級定義
- 驗證可以是同步或非同步（例如透過 API 呼叫）

## 何時執行驗證？

由您決定！`<Field />` 元件接受一些回調函數作為 props，例如 `onChange` 或 `onBlur`。這些回調會接收欄位當前值和 fieldAPI 物件，讓您執行驗證。如果發現驗證錯誤，只需回傳錯誤訊息字串，該訊息就會出現在 `field.state.meta.errors` 中。

以下是一個範例：

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

在上面的範例中，驗證在每次鍵盤輸入時執行（`onChange`）。如果我們想改為在欄位失焦時驗證，可以這樣修改程式碼：

```tsx
<form.Field
  name="age"
  validators={{
    onBlur: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        // 監聽欄位的 onBlur 事件
        onBlur={field.handleBlur}
        // 我們仍需實作 onChange，讓 TanStack Form 能接收變更
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

透過實作不同的回調函數，您可以控制驗證的執行時機。您甚至可以在不同時間點執行不同的驗證邏輯：

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
    onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        // 監聽欄位的 onBlur 事件
        onBlur={field.handleBlur}
        // 我們仍需實作 onChange，讓 TanStack Form 能接收變更
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

在上面的範例中，我們對同一個欄位在不同時間點（每次鍵盤輸入和欄位失焦時）執行不同的驗證。由於 `field.state.meta.errors` 是一個陣列，所有相關錯誤訊息都會在適當時機顯示。您也可以使用 `field.state.meta.errorMap` 根據驗證執行時機（onChange、onBlur 等）取得錯誤訊息。更多關於錯誤顯示的資訊請見下文。

## 顯示錯誤訊息

設定好驗證後，您可以將錯誤訊息從陣列映射到 UI 中顯示：

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => {
    return (
      <>
        {/* ... */}
        {!field.state.meta.isValid && (
          <em>{field.state.meta.errors.join(',')}</em>
        )}
      </>
    )
  }}
</form.Field>
```

或者使用 `errorMap` 屬性來存取特定錯誤訊息：

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => (
    <>
      {/* ... */}
      {field.state.meta.errorMap['onChange'] ? (
        <em>{field.state.meta.errorMap['onChange']}</em>
      ) : null}
    </>
  )}
</form.Field>
```

值得注意的是，我們的 `errors` 陣列和 `errorMap` 會匹配驗證器回傳的型別。這意味著：

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }}
>
  {(field) => (
    <>
      {/* ... */}
      {/* errorMap.onChange 的型別是 `{isOldEnough: false} | undefined` */}
      {/* meta.errors 的型別是 `Array<{isOldEnough: false} | undefined>` */}
      {!field.state.meta.errorMap['onChange']?.isOldEnough ? (
        <em>The user is not old enough</em>
      ) : null}
    </>
  )}
</form.Field>
```

## 欄位層級與表單層級驗證

如上所示，每個 `<Field>` 都可以透過 `onChange`、`onBlur` 等回調函數接受自己的驗證規則。也可以透過將類似的回調傳遞給 `useForm()` hook，在表單層級（而不是逐個欄位）定義驗證規則。

範例：

```tsx
export default function App() {
  const form = useForm({
    defaultValues: {
      age: 0,
    },
    onSubmit: async ({ value }) => {
      console.log(value)
    },
    validators: {
      // 以與欄位相同的方式向表單添加驗證器
      onChange({ value }) {
        if (value.age < 13) {
          return 'Must be 13 or older to sign'
        }
        return undefined
      },
    },
  })

  // 訂閱表單的 errorMap，以便更新時重新渲染
  // 或者，您可以使用 `form.Subscribe`
  const formErrorMap = useStore(form.store, (state) => state.errorMap)

  return (
    <div>
      {/* ... */}
      {formErrorMap.onChange ? (
        <div>
          <em>There was an error on the form: {formErrorMap.onChange}</em>
        </div>
      ) : null}
      {/* ... */}
    </div>
  )
}
```

### 從表單驗證器設置欄位級錯誤

您可以從表單驗證器中設置欄位錯誤。一個常見的使用情境是在表單的 `onSubmitAsync` 驗證器中呼叫單一 API 端點來驗證所有欄位。

```tsx
export default function App() {
  const form = useForm({
    defaultValues: {
      age: 0,
      socials: [],
      details: {
        email: '',
      },
    },
    validators: {
      onSubmitAsync: async ({ value }) => {
        // 在伺服器上驗證值
        const hasErrors = await verifyDataOnServer(value)
        if (hasErrors) {
          return {
            form: 'Invalid data', // `form` 鍵是可選的
            fields: {
              age: 'Must be 13 or older to sign',
              // 使用欄位名稱設置嵌套欄位的錯誤
              'socials[0].url': 'The provided URL does not exist',
              'details.email': 'An email is required',
            },
          }
        }

        return null
      },
    },
  })

  return (
    <div>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          void form.handleSubmit()
        }}
      >
        <form.Field name="age">
          {(field) => (
            <>
              <label htmlFor={field.name}>Age:</label>
              <input
                id={field.name}
                name={field.name}
                value={field.state.value}
                type="number"
                onChange={(e) => field.handleChange(e.target.valueAsNumber)}
              />
              {!field.state.meta.isValid && (
                <em role="alert">{field.state.meta.errors.join(', ')}</em>
              )}
            </>
          )}
        </form.Field>
        <form.Subscribe
          selector={(state) => [state.errorMap]}
          children={([errorMap]) =>
            errorMap.onSubmit ? (
              <div>
                <em>There was an error on the form: {errorMap.onSubmit}</em>
              </div>
            ) : null
          }
        />
        {/*...*/}
      </form>
    </div>
  )
}
```

> 值得注意的是，如果表單驗證函數回傳錯誤，該錯誤可能會被欄位特定的驗證覆寫。
>
> 這意味著：
>
> ```jsx
> const form = useForm({
>   defaultValues: {
>     age: 0,
>   },
>   validators: {
>     onChange: ({ value }) => {
>       return {
>         fields: {
>           age: value.age < 12 ? 'Too young!' : undefined,
>         },
>       }
>     },
>   },
> })
>
> // ...
>
> return (
>   <form.Field
>     name="age"
>     validators={{
>       onChange: ({ value }) => (value % 2 === 0 ? 'Must be odd!' : undefined),
>     }}
>     children={() => <>{/* ... */}</>}
>   />
> )
> ```
>
> 即使表單級驗證回傳了 'Too young!' 錯誤，也只會顯示 `'Must be odd!'`。

## 非同步函數驗證

雖然我們預期大多數驗證會是同步的，但在許多情況下，透過網路呼叫或其他非同步操作進行驗證會很有用。

為此，我們提供了專用的 `onChangeAsync`、`onBlurAsync` 等方法：

```tsx
<form.Field
  name="age"
  validators={{
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value < 13 ? 'You must be 13 to make an account' : undefined
    },
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

同步和非同步驗證可以共存。例如，可以在同一個欄位上同時定義 `onBlur` 和 `onBlurAsync`：

```tsx
<form.Field
  name="age"
  validators={{
    onBlur: ({ value }) => (value < 13 ? 'You must be at least 13' : undefined),
    onBlurAsync: async ({ value }) => {
      const currentAge = await fetchCurrentAgeOnProfile()
      return value < currentAge ? 'You can only increase the age' : undefined
    },
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

同步驗證方法（`onBlur`）會先執行，只有當同步驗證成功時，才會執行非同步方法（`onBlurAsync`）。要改變此行為，請將 `asyncAlways` 選項設為 `true`，這樣無論同步方法的結果如何，都會執行非同步方法。

### 內建防抖動 (Debouncing)

雖然非同步呼叫是驗證資料庫的有效方式，但在每次鍵盤輸入時發送網路請求可能會導致資料庫遭受 DDoS 攻擊。

相反地，我們提供了一個簡單的方法來防抖動您的 `async` 呼叫，只需添加一個屬性：

```tsx
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsync: async ({ value }) => {
      // ...
    },
  }}
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

這將以 500 毫秒的延遲防抖動每個非同步呼叫。您甚至可以針對每個驗證屬性覆寫此設定：

```tsx
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsyncDebounceMs: 1500,
    onChangeAsync: async ({ value }) => {
      // ...
    },
    onBlurAsync: async ({ value }) => {
      // ...
    },
  }}
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

這將使 `onChangeAsync` 每 1500 毫秒執行一次，而 `onBlurAsync` 則每 500 毫秒執行一次。

## 透過 Schema 函式庫驗證

雖然函數提供了更多靈活性和自訂驗證的能力，但有時可能會顯得冗長。為了解決這個問題，有一些函式庫提供了基於 schema 的驗證，使簡寫和型別嚴格的驗證變得更加容易。您還可以為整個表單定義單一 schema 並將其傳遞給表單層級，錯誤將自動傳播到各個欄位。

### 標準 Schema 函式庫

TanStack Form 原生支援所有遵循 [Standard Schema 規範](https://github.com/standard-schema/standard-schema) 的函式庫，最著名的包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)
- [Effect/Schema](https://effect.website/docs/schema/standard-schema/)

_注意：_ 請確保使用 schema 函式庫的最新版本，舊版本可能尚未支援 Standard Schema。

> 驗證不會提供轉換後的值。更多資訊請參閱 [提交處理](./submission-handling.md)。

要使用這些函式庫的 schema，您可以像使用自訂函數一樣將其傳遞給 `validators` props：

```tsx
const userSchema = z.object({
  age: z.number().gte(13, 'You must be 13 to make an account'),
})

function App() {
  const form = useForm({
    defaultValues: {
      age: 0,
    },
    validators: {
      onChange: userSchema,
    },
  })
  return (
    <div>
      <form.Field
        name="age"
        children={(field) => {
          return <>{/* ... */}</>
        }}
      />
    </div>
  )
}
```

表單和欄位層級的非同步驗證也同樣支援：

```tsx
<form.Field
  name="age"
  validators={{
    onChange: z.number().gte(13, 'You must be 13 to make an account'),
    onChangeAsyncDebounceMs: 500,
    onChangeAsync: z.number().refine(
      async (value) => {
        const currentAge = await fetchCurrentAgeOnProfile()
        return value >= currentAge
      },
      {
        message: 'You can only increase the age',
      },
    ),
  }}
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

如果您需要對 Standard Schema 驗證有更多控制，可以像這樣將 Standard Schema 與回調函數結合使用：

```tsx
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsync: async ({ value, fieldApi }) => {
      const errors = fieldApi.parseValueWithSchema(
        z.number().gte(13, 'You must be 13 to make an account'),
      )
      if (errors) return errors
      // 繼續您的驗證
    },
  }}
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

## 防止提交無效表單

`onChange`、`onBlur` 等回調也會在表單提交時執行，如果表單無效，提交將被阻止。

表單狀態物件有一個 `canSubmit` 標誌，當任何欄位無效且表單已被觸碰時，該標誌為 false（`canSubmit` 在表單被觸碰前為 true，即使某些欄位根據其 `onChange`/`onBlur` props「技術上」無效）。

您可以透過 `form.Subscribe` 訂閱它，並使用該值來實現例如在表單無效時禁用提交按鈕（實際上，禁用按鈕不可訪問，請改用 `aria-disabled`）。

```tsx
const form = useForm(/* ... */)

return (
  /* ... */

  // 動態提交按鈕
  <form.Subscribe
    selector={(state) => [state.canSubmit, state.isSubmitting]}
    children={([canSubmit, isSubmitting]) => (
      <button type="submit" disabled={!canSubmit}>
        {isSubmitting ? '...' :
```
