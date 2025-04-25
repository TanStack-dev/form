---
source-updated-at: '2025-04-16T14:15:06.000Z'
translation-updated-at: '2025-04-25T20:34:32.838Z'
id: form-validation
title: 表單驗證
---
## 表單與欄位驗證 (Form and Field Validation)

TanStack Form 的核心功能之一就是驗證機制。TanStack Form 讓驗證功能高度可自訂：

- 您可以控制驗證的時機（在變更時、輸入時、失去焦點時、提交時...）
- 驗證規則可以在欄位層級或表單層級定義
- 驗證可以是同步或非同步的（例如 API 呼叫的結果）

## 驗證何時執行？

由您決定！`<Field />` 元件接受一些回調函式作為 props，例如 `onChange` 或 `onBlur`。這些回調會接收欄位的當前值以及 fieldAPI 物件，讓您可以執行驗證。如果發現驗證錯誤，只需返回錯誤訊息字串，它就會出現在 `field.state.meta.errors` 中。

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
      {field.state.meta.errors ? (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

在上面的範例中，驗證在每次按鍵時執行（`onChange`）。如果我們希望驗證在欄位失去焦點時執行，可以這樣修改程式碼：

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
        // 我們始終需要實作 onChange，以便 TanStack Form 接收變更
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {field.state.meta.errors ? (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

因此，您可以透過實作所需的回調來控制驗證的時機。您甚至可以在不同時間執行不同的驗證：

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
        // 我們始終需要實作 onChange，以便 TanStack Form 接收變更
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {field.state.meta.errors ? (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

在上面的範例中，我們在同一欄位的不同時間（每次按鍵和失去焦點時）驗證不同內容。由於 `field.state.meta.errors` 是一個陣列，所有相關錯誤都會在給定時間顯示。您也可以使用 `field.state.meta.errorMap` 根據驗證執行的時間（onChange、onBlur 等）取得錯誤。更多關於顯示錯誤的資訊如下。

## 顯示錯誤

一旦設定好驗證，您可以將錯誤從陣列映射到 UI 中顯示：

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
        {field.state.meta.errors.length ? (
          <em>{field.state.meta.errors.join(',')}</em>
        ) : null}
      </>
    )
  }}
</form.Field>
```

或者使用 `errorMap` 屬性來存取特定錯誤：

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

值得一提的是，我們的 `errors` 陣列和 `errorMap` 與驗證器返回的類型匹配。這意味著：

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
      {/* errorMap.onChange 的類型是 `{isOldEnough: false} | undefined` */}
      {/* meta.errors 的類型是 `Array<{isOldEnough: false} | undefined>` */}
      {!field.state.meta.errorMap['onChange']?.isOldEnough ? (
        <em>The user is not old enough</em>
      ) : null}
    </>
  )}
</form.Field>
```

## 欄位層級與表單層級的驗證

如上所示，每個 `<Field>` 透過 `onChange`、`onBlur` 等回調接受自己的驗證規則。也可以透過將類似的回調傳遞給 `useForm()` 鉤子來定義表單層級的驗證規則（而不是逐個欄位定義）。

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

  // 訂閱表單的錯誤映射，以便更新時重新渲染
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

### 從表單驗證器設置欄位層級錯誤

您可以從表單的驗證器中設置欄位錯誤。一個常見的用例是在表單的 `onSubmitAsync` 驗證器中呼叫單個 API 端點來驗證所有欄位。

```tsx
export default function App() {
  const form = useForm({
    defaultValues: {
      age: 0,
    },
    validators: {
      onSubmitAsync: async ({ value }) => {
        // 在伺服器上驗證年齡
        const isOlderThan13 = await verifyAgeOnServer(value.age)
        if (!isOlderThan13) {
          return {
            form: 'Invalid data', // `form` 鍵是可選的
            fields: {
              age: 'Must be 13 or older to sign',
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
              {field.state.meta.errors ? (
                <em role="alert">{field.state.meta.errors.join(', ')}</em>
              ) : null}
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

> 值得一提的是，如果您有一個返回錯誤的表單驗證函數，該錯誤可能會被欄位特定的驗證覆蓋。
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
>   />
> )
> ```
>
> 即使表單層級驗證返回了 'Too young!' 錯誤，也只會顯示 `'Must be odd!'`。

## 非同步函數驗證

雖然我們認為大多數驗證會是同步的，但在許多情況下，進行網路呼叫或其他非同步操作來驗證會很有用。

為此，我們提供了專用的 `onChangeAsync`、`onBlurAsync` 等方法來進行驗證：

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
      {field.state.meta.errors ? (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

同步和非同步驗證可以共存。例如，可以在同一欄位上同時定義 `onBlur` 和 `onBlurAsync`：

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
      {field.state.meta.errors ? (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

同步驗證方法（`onBlur`）會先執行，非同步方法（`onBlurAsync`）只有在同步方法（`onBlur`）成功時才會執行。要改變這種行為，請將 `asyncAlways` 選項設為 `true`，這樣無論同步方法的結果如何，非同步方法都會執行。

### 內建防抖動 (Debouncing)

雖然非同步呼叫是在驗證資料庫時的好方法，但在每次按鍵時發送網路請求是對資料庫進行 DDOS 攻擊的好方法。

相反，我們提供了一種簡單的方法來防抖動您的 `async` 呼叫，只需添加一個屬性：

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

這將以 500 毫秒的延遲防抖動每個非同步呼叫。您甚至可以針對每個驗證屬性覆蓋此屬性：

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

> 這將每 1500 毫秒執行一次 `onChangeAsync`，而 `onBlurAsync` 將每 500 毫秒執行一次。

## 透過結構描述 (Schema) 函式庫進行驗證

雖然函數提供了更多靈活性和自訂驗證的能力，但它們可能有點冗長。為了解決這個問題，有一些函式庫提供了基於結構描述的驗證，使簡寫和類型嚴格的驗證變得更加容易。您還可以為整個表單定義一個結構描述並將其傳遞給表單層級，錯誤將自動傳播到欄位。

### 標準結構描述 (Standard Schema) 函式庫

TanStack Form 原生支援所有遵循 [Standard Schema 規範](https://github.com/standard-schema/standard-schema) 的函式庫，最著名的包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)
- [Effect/Schema](https://effect.website/docs/schema/standard-schema/)

_注意：_ 請確保使用結構描述函式庫的最新版本，因為舊版本可能還不支援 Standard Schema。

要使用這些函式庫的結構描述，您可以像使用自訂函數一樣將它們傳遞給 `validators` props：

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

表單和欄位層級的非同步驗證也受到支援：

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

如果您需要對 Standard Schema 驗證進行更多控制，可以像這樣將 Standard Schema 與回調函數結合使用：

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
      // 繼續進行您的驗證
    },
  }}
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

## 防止提交無效表單

`onChange`、`onBlur` 等回調也會在提交表單時執行，如果表單無效，提交將被阻止。

表單狀態物件有一個 `canSubmit` 標誌，當任何欄位無效且表單已被觸碰時為 false（`canSubmit` 在表單被觸碰之前為 true，即使某些欄位根據其 `onChange`/`onBlur` props 在「技術上」無效）。

您可以透過 `form.Subscribe` 訂閱它，並使用該值來實現例如在表單無效時禁用提交按鈕（實際上，禁用的按鈕不可訪問，請改用 `aria-disabled`）。

```tsx
const form = useForm(/* ... */)

return (
  /* ... */

  // 動態提交按鈕
  <form.Subscribe
    selector={(state) => [state.canSubmit, state.isSubmitting]}
    children={([canSubmit, isSubmitting]) => (
      <button type="submit" disabled={!canSubmit}>
        {isSubmitting ? '...' : 'Submit'}
      </button>
    )}
  />
)
```
