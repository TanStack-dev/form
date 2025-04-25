---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-25T20:48:34.853Z'
id: basic-concepts
title: 基本概念
---
本頁介紹 `@tanstack/svelte-form` 函式庫中使用的基本概念與術語。熟悉這些概念將幫助您更有效地理解與使用此函式庫。

## 表單選項 (Form Options)

您可以使用 `formOptions` 函式建立表單選項，以便在多個表單之間共享配置。

範例：

```ts
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## 表單實例 (Form Instance)

表單實例 (Form Instance) 是一個代表單一表單的物件，提供操作表單的方法與屬性。您可以使用 `createForm` 函式建立表單實例。此函式接受包含 `onSubmit` 函式的物件，該函式會在表單提交時被呼叫。

```ts
const form = createForm(() => ({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // 處理表單資料
    console.log(value)
  },
}))
```

您也可以不使用 `formOptions` 直接建立表單實例：

```ts
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

欄位 (Field) 代表單一表單輸入元素，例如文字輸入框或核取方塊。欄位是透過表單實例提供的 `form.Field` 元件建立。該元件接受 `name` 屬性 (應與表單預設值中的鍵名匹配) 和 `children` 屬性 (這是一個接收欄位物件作為參數的渲染函式)。

範例：

```svelte
<form.Field name="firstName">
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onblur={field.handleBlur}
      oninput={(e) => field.handleChange(e.target.value)}
    />
  {/snippet}
</form.Field>
```

## 欄位狀態 (Field State)

每個欄位都有自身的狀態，包含當前值、驗證狀態、錯誤訊息和其他元資料。您可以透過 `field.state` 屬性存取欄位狀態。

範例：

```ts
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

有三種欄位狀態對於觀察使用者互動特別有用：當使用者點擊/切換到欄位時會標記為 _"touched"_、在值被修改前保持 _"pristine"_、值變更後則標記為 _"dirty"_。您可以透過以下標誌檢查這些狀態：

```ts
const { isTouched, isPristine, isDirty } = field.state.meta
```

![欄位狀態](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## 欄位 API (Field API)

欄位 API (Field API) 是建立欄位時傳遞給渲染函式的物件，提供操作欄位狀態的方法。

範例：

```svelte
<input
  name={field.name}
  value={field.state.value}
  onblur={field.handleBlur}
  oninput={(e) => field.handleChange(e.target.value)}
/>
```

## 驗證 (Validation)

`@tanstack/svelte-form` 原生支援同步與非同步驗證。驗證函式可透過 `validators` 屬性傳遞給 `form.Field` 元件。

範例：

```svelte
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
>
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onBlur={field.handleBlur}
      onInput={(e) => field.handleChange(e.target.value)}
    />
    <p>{field.state.meta.errors[0]}</p>
  {/snippet}
</form.Field>
```

## 使用標準結構描述函式庫驗證 (Validation with Standard Schema Libraries)

除了手動驗證選項外，我們也支援 [Standard Schema](https://github.com/standard-schema/standard-schema) 規範。

您可以使用任何實作此規範的函式庫定義結構描述，並將其傳遞給表單或欄位驗證器。

支援的函式庫包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```svelte
<script>
  import { z } from 'zod'

  // ...
</script>

<form.Field
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
>
  {#snippet children(field)}
    <input
      name={field.name}
      value={field.state.value}
      onBlur={field.handleBlur}
      onInput={(e) => field.handleChange(e.target.value)}
    />
    <p>{field.state.meta.errors[0]}</p>
  {/snippet}
</form.Field>
```

## 響應式 (Reactivity)

`@tanstack/svelte-form` 提供多種訂閱表單與欄位狀態變更的方式，最值得注意的是 `form.useStore` 鉤子 (hook) 和 `form.Subscribe` 元件。這些方法讓您可以透過僅在必要時更新元件來優化表單的渲染效能。

範例：

```svelte
<script>
  //...
  const firstName = form.useStore((state) => state.values.firstName)
</script>

<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
>
  {#snippet children(state)}
    <button type="submit" disabled={!state.canSubmit}>
      {state.isSubmitting ? '...' : '提交'}
    </button>
  {/snippet}
</form.Subscribe>
```

## 陣列欄位 (Array Fields)

陣列欄位 (Array Fields) 讓您能管理表單中的值列表，例如興趣愛好清單。您可以使用 `form.Field` 元件並設定 `mode="array"` 屬性來建立陣列欄位。

操作陣列欄位時，可使用 `pushValue`、`removeValue`、`swapValues` 和 `moveValue` 方法來新增、移除和交換陣列中的值。

範例：

```svelte
<form.Field name="hobbies" mode="array">
  {#snippet children(hobbiesField)}
    <div>
      興趣愛好
      <div>
        {#each hobbiesField.state.value as _, i}
            <div>
              <form.Field name={`hobbies[${i}].name`}>
                {#snippet children(field)}
                  <div>
                    <label for={field.name}>名稱：</label>
                    <input
                      id={field.name}
                      name={field.name}
                      value={field.state.value}
                      onblur={field.handleBlur}
                      onchange={(e) => field.handleChange(e.target.value)}
                    />
                    <button
                      type="button"
                      onclick={() => hobbiesField.removeValue(i)}
                    >
                      X
                    </button>
                  </div>
                {/snippet}
              </form.Field>
              <form.Field name={`hobbies[${i}].description`}>
                {#snippet children(field)}
                    <div>
                      <label for={field.name}>描述：</label>
                      <input
                        id={field.name}
                        name={field.name}
                        value={field.state.value}
                        onblur={field.handleBlur}
                        onchange={(e) => field.handleChange(e.target.value)}
                      />
                    </div>
                {/snippet}
              </form.Field>
            </div>
          {:else}
            未找到興趣愛好。
          {/each}
      </div>
      <button
        type="button"
        onclick={() =>
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
  {/snippet}
</form.Field>
```

以上是 `@tanstack/svelte-form` 函式庫中使用的基本概念與術語。理解這些概念將幫助您更有效地使用此函式庫，輕鬆建立複雜的表單。
