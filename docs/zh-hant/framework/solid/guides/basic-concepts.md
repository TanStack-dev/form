---
source-updated-at: '2025-03-14T21:54:05.000Z'
translation-updated-at: '2025-04-25T20:48:36.269Z'
id: basic-concepts
title: 基本概念
---

本頁介紹 `@tanstack/solid-form` 函式庫中使用的基本概念與術語。熟悉這些概念將幫助您更好地理解並運用此函式庫。

## 表單選項 (Form Options)

您可以使用 `formOptions` 函式建立表單選項，以便在多個表單之間共享配置。

範例：

```tsx
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## 表單實例 (Form Instance)

表單實例是一個代表單一表單的物件，提供操作表單的方法與屬性。您可以使用表單選項提供的 `createForm` 鉤子來建立表單實例。該鉤子接受包含 `onSubmit` 函式的物件，此函式會在表單提交時被呼叫。

```tsx
const form = createForm(() => ({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // 處理表單資料
    console.log(value)
  },
}))
```

您也可以不使用 `formOptions`，直接透過獨立的 `createForm` API 建立表單實例：

```tsx
const form = createForm<Person>(() => ({
  onSubmit: async ({ value }) => {
    // 處理表單資料
    console.log(value)
  },
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  },
}))
```

## 欄位 (Field)

欄位代表單一表單輸入元素，例如文字輸入框或核取方塊。欄位是透過表單實例提供的 `form.Field` 元件建立的。該元件接受一個名稱屬性 (name prop)，應與表單預設值中的鍵名匹配。同時也接受子元件屬性 (children prop)，這是一個以欄位物件為參數的渲染函式。

範例：

```tsx
<form.Field
  name="firstName"
  children={(field) => (
    <input
      name={field().name}
      value={field().state.value}
      onBlur={field().handleBlur}
      onInput={(e) => field().handleChange(e.target.value)}
    />
  )}
/>
```

## 欄位狀態 (Field State)

每個欄位都有自身的狀態，包含當前值、驗證狀態、錯誤訊息和其他元資料。您可以透過 `field().state` 屬性存取欄位狀態。

範例：

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field().state
```

有三種欄位狀態對了解使用者互動特別有用：當使用者點擊/切換到欄位時會標記為「已觸碰 _(touched)_」；在值被改變前保持「原始 _(pristine)_」狀態；值改變後則標記為「已修改 _(dirty)_」。您可以透過以下標誌檢查這些狀態：

```tsx
const { isTouched, isPristine, isDirty } = field().state.meta
```

![欄位狀態](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## 欄位 API (Field API)

欄位 API 是建立欄位時傳遞給渲染函式的物件，提供操作欄位狀態的方法。

範例：

```tsx
<input
  name={field().name}
  value={field().state.value}
  onBlur={field().handleBlur}
  onInput={(e) => field().handleChange(e.target.value)}
/>
```

## 驗證 (Validation)

`@tanstack/solid-form` 原生支援同步與非同步驗證。驗證函式可以透過 `validators` 屬性傳遞給 `form.Field` 元件。

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
        name={field().name}
        value={field().state.value}
        onBlur={field().handleBlur}
        onInput={(e) => field().handleChange(e.target.value)}
      />
      <p>{field().state.meta.errors[0]}</p>
    </>
  )}
/>
```

## 使用標準結構描述函式庫驗證 (Validation with Standard Schema Libraries)

除了手動驗證選項外，我們也支援 [標準結構描述 (Standard Schema)](https://github.com/standard-schema/standard-schema) 規範。

您可以使用任何實現此規範的函式庫定義結構描述，並將其傳遞給表單或欄位驗證器。

支援的函式庫包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```tsx
import { z } from 'zod'

// ...
;<form.Field
  name="firstName"
  validators={{
    onChange: z.string().min(3, '名字至少需要 3 個字元'),
    onChangeAsyncDebounceMs: 500,
    onChangeAsync: z.string().refine(
      async (value) => {
        await new Promise((resolve) => setTimeout(resolve, 1000))
        return !value.includes('error')
      },
      {
        message: '名字中不能包含 "error"',
      },
    ),
  }}
  children={(field) => (
    <>
      <input
        name={field().name}
        value={field().state.value}
        onBlur={field().handleBlur}
        onInput={(e) => field().handleChange(e.target.value)}
      />
      <p>{field().state.meta.errors[0]}</p>
    </>
  )}
/>
```

## 響應式 (Reactivity)

`@tanstack/solid-form` 提供多種訂閱表單與欄位狀態變更的方式，最值得注意的是 `form.useStore` 鉤子與 `form.Subscribe` 元件。這些方法讓您可以透過只在必要時更新元件來優化表單的渲染效能。

範例：

```tsx
const firstName = form.useStore((state) => state.values.firstName)
//...
<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
  children={(state) => (
    <button type="submit" disabled={!state().canSubmit}>
      {state().isSubmitting ? '...' : '提交'}
    </button>
  )}
/>
```

## 陣列欄位 (Array Fields)

陣列欄位讓您能管理表單中的值列表，例如興趣愛好清單。您可以透過 `form.Field` 元件並設定 `mode="array"` 屬性來建立陣列欄位。

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
        <Show
          when={hobbiesField().state.value.length > 0}
          fallback={'沒有找到興趣愛好。'}
        >
          <Index each={hobbiesField().state.value}>
            {(_, i) => (
              <div>
                <form.Field
                  name={`hobbies[${i}].name`}
                  children={(field) => (
                    <div>
                      <label for={field().name}>名稱：</label>
                      <input
                        id={field().name}
                        name={field().name}
                        value={field().state.value}
                        onBlur={field().handleBlur}
                        onInput={(e) => field().handleChange(e.target.value)}
                      />
                      <button
                        type="button"
                        onClick={() => hobbiesField().removeValue(i)}
                      >
                        X
                      </button>
                    </div>
                  )}
                />
                <form.Field
                  name={`hobbies[${i}].description`}
                  children={(field) => {
                    return (
                      <div>
                        <label for={field().name}>描述：</label>
                        <input
                          id={field().name}
                          name={field().name}
                          value={field().state.value}
                          onBlur={field().handleBlur}
                          onInput={(e) => field().handleChange(e.target.value)}
                        />
                      </div>
                    )
                  }}
                />
              </div>
            )}
          </Index>
        </Show>
      </div>
      <button
        type="button"
        onClick={() =>
          hobbiesField().pushValue({
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

以上是 `@tanstack/solid-form` 函式庫中使用的基本概念與術語。理解這些概念將幫助您更有效率地運用此函式庫，輕鬆建立複雜的表單。
