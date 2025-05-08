---
source-updated-at: '2025-05-08T07:42:29.000Z'
translation-updated-at: '2025-05-08T23:43:48.828Z'
id: basic-concepts
title: 基本概念
---

本頁介紹 `@tanstack/react-form` 函式庫中使用的基本概念與術語。熟悉這些概念將幫助您更有效地理解並運用此函式庫。

## 表單選項 (Form Options)

您可以使用 `formOptions` 函式建立表單選項，以便在多個表單之間共享配置。

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

表單實例是一個代表個別表單的物件，提供操作表單的方法與屬性。您可以使用表單選項提供的 `useForm` 鉤子 (hook) 來建立表單實例。此鉤子接受包含 `onSubmit` 函式的物件，該函式會在表單提交時被呼叫。

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

欄位代表單一表單輸入元素，例如文字輸入框或核取方塊。欄位是透過表單實例提供的 `form.Field` 元件建立的。此元件接受 `name` 屬性，應與表單預設值中的鍵名相符。同時也接受 `children` 屬性，這是一個渲染函式 (render prop)，接收欄位物件作為參數。

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

每個欄位都有自己的狀態，包含當前值、驗證狀態、錯誤訊息及其他元資料。您可以透過 `field.state` 屬性存取欄位狀態。

範例：

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

在元資料中有三個狀態對於觀察使用者與欄位的互動特別有用：

- _"isTouched"_：當使用者點擊/切換到該欄位後
- _"isPristine"_：直到使用者改變欄位值之前
- _"isDirty"_：當欄位值被改變後

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![欄位狀態](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## 理解不同函式庫中的 'isDirty'

非持久性 `dirty` 狀態

- **函式庫**：React Hook Form (RHF)、Formik、Final Form。
- **行為**：當欄位值與預設值不同時視為 'dirty'。若恢復為預設值則變回 'clean'。

持久性 `dirty` 狀態

- **函式庫**：Angular Form、Vue FormKit。
- **行為**：欄位一旦被改變即保持 'dirty' 狀態，即使恢復為預設值。

我們選擇採用持久性 'dirty' 狀態模型。為了同時支援非持久性 'dirty' 狀態，我們引入了 `isDefault` 標記。此標記可作為非持久性 'dirty' 狀態的反向指標。

```tsx
const { isTouched, isPristine, isDirty, isDefaultValue } = field.state.meta

// 以下程式碼可重現非持久性 `dirty` 功能
const nonPersistentIsDirty = !isDefaultValue
```

![延伸欄位狀態](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states-extended.png)

## 欄位 API (Field API)

欄位 API 是在建立欄位時傳遞給渲染函式的物件，提供操作欄位狀態的方法。

範例：

```tsx
<input
  value={field.state.value}
  onBlur={field.handleBlur}
  onChange={(e) => field.handleChange(e.target.value)}
/>
```

## 驗證 (Validation)

`@tanstack/react-form` 提供同步與非同步驗證功能。驗證函式可透過 `validators` 屬性傳遞給 `form.Field` 元件。

範例：

```tsx
<form.Field
  name="firstName"
  validators={{
    onChange: ({ value }) =>
      !value
        ? '必須填寫名字'
        : value.length < 3
          ? '名字至少需要 3 個字元'
          : undefined,
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && '名字中不能包含 "error"'
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

除了手動編寫驗證選項外，我們也支援 [Standard Schema](https://github.com/standard-schema/standard-schema) 規範。

您可以使用任何實現此規範的函式庫定義結構描述 (schema)，並將其傳遞給表單或欄位驗證器。

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

`@tanstack/react-form` 提供多種訂閱表單與欄位狀態變更的方式，最值得注意的是 `useStore(form.store)` 鉤子與 `form.Subscribe` 元件。這些方法讓您可以最佳化表單的渲染效能，只在必要時更新元件。

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

請注意，雖然 `useStore` 鉤子的 `selector` 屬性是選填的，但強烈建議提供此屬性，因為省略它會導致不必要的重新渲染。

```tsx
// 正確用法
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
// 錯誤用法
const store = useStore(form.store)
```

注意：不建議使用 `useField` 鉤子來實現響應式功能，因為它設計上是為了在 `form.Field` 元件中謹慎使用。您應該考慮改用 `useStore(form.store)`。

## 監聽器 (Listeners)

`@tanstack/react-form` 允許您對特定觸發事件做出反應並「監聽」這些事件以執行副作用。

範例：

```tsx
<form.Field
  name="country"
  listeners={{
    onChange: ({ value }) => {
      console.log(`國家已變更為：${value}，正在重設省份`)
      form.setFieldValue('province', '')
    },
  }}
/>
```

更多資訊請參閱 [監聽器](./listeners.md)

## 陣列欄位 (Array Fields)

陣列欄位讓您能管理表單中的值列表，例如興趣愛好清單。您可以使用 `form.Field` 元件並設定 `mode="array"` 屬性來建立陣列欄位。

操作陣列欄位時，可以使用欄位的 `pushValue`、`removeValue`、`swapValues` 和 `moveValue` 方法來新增、刪除和交換陣列中的值。

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
          ? '沒有找到興趣愛好。'
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
        新增興趣
      </button>
    </div>
  )}
/>
```

## 重設按鈕 (Reset Buttons)

當在 TanStack Form 中使用 `<button type="reset">` 與 `form.reset()` 時，您需要阻止預設的 HTML 重設行為，以避免表單元素（特別是 `<select>` 元素）意外重設為其初始 HTML 值。在按鈕的 `onClick` 處理程式中使用 `event.preventDefault()` 來阻止原生表單重設。

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

或者，您也可以使用 `<button type="button">` 來避免原生的 HTML 重設行為。

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

以上是 `@tanstack/react-form` 函式庫中使用的基本概念與術語。理解這些概念將幫助您更有效地運用此函式庫，輕鬆建立複雜的表單。
