---
source-updated-at: '2025-04-16T08:45:06.000Z'
translation-updated-at: '2025-04-25T20:33:31.771Z'
id: basic-concepts
title: 基本概念
---

本頁介紹 `@tanstack/react-form` 函式庫中使用的基本概念與術語。熟悉這些概念將幫助您更有效地理解並運用此函式庫。

## 表單選項 (Form Options)

您可以使用 `formOptions` 函式建立表單選項，以便在多個表單間共享設定。

範例：

```tsx
interface User {
  firstName: string
  lastName: string
  hobbies: Array<string>
}
const defaultUser: User = { firstName: '', lastName: '', hobbies: [] }

const formOpts = formOptions({
  defaultValues: defaultUser,
})
```

## 表單實例 (Form Instance)

表單實例是一個代表單一表單的物件，提供操作表單的方法與屬性。您可透過表單選項提供的 `useForm` 鉤子來建立表單實例。此鉤子接受包含 `onSubmit` 函式的物件，該函式會在表單提交時被呼叫。

```tsx
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // 處理表單資料
    console.log(value)
  },
})
```

您也可以不使用 `formOptions`，直接透過獨立的 `useForm` API 建立表單實例：

```tsx
interface User {
  firstName: string
  lastName: string
  hobbies: Array<string>
}
const defaultUser: User = { firstName: '', lastName: '', hobbies: [] }

const form = useForm({
  defaultValues: defaultUser,
  onSubmit: async ({ value }) => {
    // 處理表單資料
    console.log(value)
  },
})
```

## 欄位 (Field)

欄位代表單一表單輸入元素，例如文字輸入框或核取方塊。欄位是透過表單實例提供的 `form.Field` 元件來建立。此元件接受 `name` 屬性，需與表單預設值中的鍵名匹配，以及 `children` 屬性（這是一個接收欄位物件作為參數的渲染函式）。

範例：

```tsx
<form.Field
  name="firstName"
  children={(field) => (
    <>
      <input
        value={field.state.value}
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.value)}
      />
      <FieldInfo field={field} />
    </>
  )}
/>
```

## 欄位狀態 (Field State)

每個欄位都有自身的狀態，包含當前值、驗證狀態、錯誤訊息及其他元資料。您可透過 `field.state` 屬性存取欄位狀態。

範例：

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

有三種欄位狀態有助於觀察使用者互動：當使用者點擊/切換至欄位時為「已觸碰 _(touched)_」、使用者未改變值前為「原始狀態 _(pristine)_」、值變更後則為「已修改 _(dirty)_」。可透過以下標誌檢查這些狀態：

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![欄位狀態](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

> **給從 `React Hook Form` 轉換的使用者重要提示**：`TanStack/form` 中的 `isDirty` 標誌與 RHF 中同名的標誌意義不同。
> 在 RHF 中，當表單值與原始值不同時 `isDirty = true`。若使用者修改表單值後又改回與預設值相同，RHF 的 `isDirty` 會是 `false`，但在 `TanStack/form` 中會是 `true`。
> `TanStack/form` 同時在表單與欄位層級暴露預設值 (`form.options.defaultValues`, `field.options.defaultValue`)，因此若需模擬 RHF 的行為，您可以自行編寫 `isDefaultValue()` 輔助函式。

## 欄位 API (Field API)

欄位 API 是建立欄位時傳遞給渲染函式的物件，提供操作欄位狀態的方法。

範例：

```tsx
<input
  value={field.state.value}
  onBlur={field.handleBlur}
  onChange={(e) => field.handleChange(e.target.value)}
/>
```

## 驗證 (Validation)

`@tanstack/react-form` 原生支援同步與非同步驗證。驗證函式可透過 `validators` 屬性傳遞給 `form.Field` 元件。

範例：

```tsx
<form.Field
  name="firstName"
  validators={{
    onChange: ({ value }) =>
      !value
        ? '必須填寫名字'
        : value.length < 3
          ? '名字至少需 3 個字元'
          : undefined,
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && '名字中不得包含 "error"'
    },
  }}
  children={(field) => (
    <>
      <input
        value={field.state.value}
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.value)}
      />
      <FieldInfo field={field} />
    </>
  )}
/>
```

## 使用標準結構描述函式庫驗證 (Validation with Standard Schema Libraries)

除了手動驗證選項外，我們也支援 [Standard Schema](https://github.com/standard-schema/standard-schema) 規範。

您可以使用任何實作此規範的函式庫定義結構描述，並將其傳遞給表單或欄位驗證器。

支援的函式庫包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```tsx
import { z } from 'zod'

const userSchema = z.object({
  age: z.number().gte(13, '必須年滿 13 歲才能建立帳戶'),
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

## 響應式 (Reactivity)

`@tanstack/react-form` 提供多種訂閱表單與欄位狀態變更的方式，最顯著的是 `useStore(form.store)` 鉤子與 `form.Subscribe` 元件。這些方法讓您能透過僅在必要時更新元件來優化表單渲染效能。

範例：

```tsx
const firstName = useStore(form.store, (state) => state.values.firstName)
//...
<form.Subscribe
  selector={(state) => [state.canSubmit, state.isSubmitting]}
  children={([canSubmit, isSubmitting]) => (
    <button type="submit" disabled={!canSubmit}>
      {isSubmitting ? '...' : '提交'}
    </button>
  )}
/>
```

需特別注意，雖然 `useStore` 鉤子的 `selector` 屬性是可選的，但強烈建議提供此參數，因為省略它會導致不必要的重新渲染。

```tsx
// 正確用法
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
// 錯誤用法
const store = useStore(form.store)
```

注意：不建議使用 `useField` 鉤子來實現響應式，因為它設計上是供 `form.Field` 元件內部謹慎使用的。您應考慮改用 `useStore(form.store)`。

## 監聽器 (Listeners)

`@tanstack/react-form` 允許您對特定觸發事件作出反應並「監聽」它們以執行副作用。

範例：

```tsx
<form.Field
  name="country"
  listeners={{
    onChange: ({ value }) => {
      console.log(`國家已變更為: ${value}, 重置省份選項`)
      form.setFieldValue('province', '')
    },
  }}
/>
```

更多資訊請參閱 [監聽器](./listeners.md)

## 陣列欄位 (Array Fields)

陣列欄位讓您能管理表單中的值列表，例如興趣愛好清單。您可透過 `form.Field` 元件加上 `mode="array"` 屬性來建立陣列欄位。

操作陣列欄位時，可使用欄位的 `pushValue`、`removeValue`、`swapValues` 和 `moveValue` 方法來新增、移除與交換陣列中的值。

範例：

```tsx
<form.Field
  name="hobbies"
  mode="array"
  children={(hobbiesField) => (
    <div>
      興趣愛好
      <div>
        {!hobbiesField.state.value.length
          ? '尚未新增任何興趣愛好。'
          : hobbiesField.state.value.map((_, i) => (
              <div key={i}>
                <form.Field
                  name={`hobbies[${i}].name`}
                  children={(field) => {
                    return (
                      <div>
                        <label htmlFor={field.name}>名稱：</label>
                        <input
                          id={field.name}
                          name={field.name}
                          value={field.state.value}
                          onBlur={field.handleBlur}
                          onChange={(e) => field.handleChange(e.target.value)}
                        />
                        <button
                          type="button"
                          onClick={() => hobbiesField.removeValue(i)}
                        >
                          X
                        </button>
                        <FieldInfo field={field} />
                      </div>
                    )
                  }}
                />
                <form.Field
                  name={`hobbies[${i}].description`}
                  children={(field) => {
                    return (
                      <div>
                        <label htmlFor={field.name}>描述：</label>
                        <input
                          id={field.name}
                          name={field.name}
                          value={field.state.value}
                          onBlur={field.handleBlur}
                          onChange={(e) => field.handleChange(e.target.value)}
                        />
                        <FieldInfo field={field} />
                      </div>
                    )
                  }}
                />
              </div>
            ))}
      </div>
      <button
        type="button"
        onClick={() =>
          hobbiesField.pushValue({
            name: '',
            description: '',
            yearsOfExperience: 0,
          })
        }
      >
        新增興趣愛好
      </button>
    </div>
  )}
/>
```

## 重設按鈕 (Reset Buttons)

當結合使用 `<button type="reset">` 與 TanStack Form 的 `form.reset()` 時，需阻止預設的 HTML 重設行為，以避免表單元素（特別是 `<select>` 元素）意外重設至初始 HTML 值。在按鈕的 `onClick` 處理程序中使用 `event.preventDefault()` 來阻止原生表單重設。

範例：

```tsx
<button
  type="reset"
  onClick={(event) => {
    event.preventDefault()
    form.reset()
  }}
>
  重設
</button>
```

或者，您可以使用 `<button type="button">` 來避免原生 HTML 重設行為。

```tsx
<button
  type="button"
  onClick={() => {
    form.reset()
  }}
>
  重設
</button>
```

以上是 `@tanstack/react-form` 函式庫中使用的基本概念與術語。理解這些概念將幫助您更有效率地運用此函式庫，輕鬆建立複雜的表單。
