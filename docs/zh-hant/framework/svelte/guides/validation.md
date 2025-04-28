---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-25T20:49:20.614Z'
id: form-validation
title: 表單驗證
---

TanStack Form 的核心功能之一是驗證 (validation) 概念。TanStack Form 讓驗證具有高度可自訂性：

- 您可以控制驗證的時機（在變更時、輸入時、失焦時、提交時...）
- 驗證規則可以在欄位 (field) 層級或表單 (form) 層級定義
- 驗證可以是同步或非同步（例如 API 呼叫的結果）

## 驗證何時執行？

由您決定！`<form.Field />` 元件接受一些回調函式作為 props，例如 `onChange` 或 `onBlur`。這些回調會接收欄位的當前值以及 fieldAPI 物件，讓您可以執行驗證。如果發現驗證錯誤，只需回傳錯誤訊息字串，該訊息就會出現在 `field.state.meta.errors` 中。

以下是一個範例：

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

在上面的範例中，驗證會在每次按鍵時執行（`onchange`）。如果我們想改為在欄位失焦時執行驗證，可以這樣修改程式碼：

```svelte
<form.Field
  name="age"
  validators={{
    onBlur: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

因此，您可以透過實作所需的回調來控制驗證的時機。您甚至可以在不同時間執行不同的驗證：

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
    onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

在上面的範例中，我們在不同時間（每次按鍵和欄位失焦時）對同一欄位進行不同的驗證。由於 `field.state.meta.errors` 是一個陣列，所有相關的錯誤都會在特定時間顯示。您也可以使用 `field.state.meta.errorMap` 根據驗證時機（onChange、onBlur 等）取得錯誤。更多關於顯示錯誤的資訊如下。

## 顯示錯誤

設定好驗證後，您可以將錯誤從陣列映射到 UI 中顯示：

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

或者使用 `errorMap` 屬性來存取特定的錯誤：

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    {#if field.state.meta.errorMap['onChange']}
      <em role="alert">{field.state.meta.errorMap['onChange']}</em>
    {/if}
  {/snippet}
</form.Field>
```

值得一提的是，我們的 `errors` 陣列和 `errorMap` 會與驗證器回傳的型別匹配。這意味著：

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    <!-- errorMap.onChange 的型別是 `{isOldEnough: false} | undefined` -->
    <!-- meta.errors 的型別是 `Array<{isOldEnough: false} | undefined>` -->
    {#if field.state.meta.errorMap['onChange']?.isOldEnough}
        <em>The user is not old enough</em>
    {/if}
  {/snippet}
</form.Field>
```

## 欄位層級與表單層級的驗證

如上所示，每個 `<form.Field>` 都透過 `onChange`、`onBlur` 等回調接受自己的驗證規則。也可以透過將類似的回調傳遞給 `createForm()` 鉤子，在表單層級（而不是逐個欄位）定義驗證規則。

範例：

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
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
  }))

  // 訂閱表單的錯誤映射，以便更新時重新渲染
  // 或者，您可以使用 `form.Subscribe`
  const formErrorMap = form.useStore((state) => state.errorMap)
</script>

<div>
  <!-- ... -->
  {#if formErrorMap.current.onChange}
    <div>
      <em>There was an error on the form: {formErrorMap.current.onChange}</em>
    </div>
  {/if}
  <!-- ... -->
</div>
```

## 非同步函式驗證

雖然我們認為大多數驗證會是同步的，但在許多情況下，透過網路呼叫或其他非同步操作進行驗證會很有用。

為此，我們提供了專用的 `onChangeAsync`、`onBlurAsync` 等方法來進行驗證：

```svelte
<form.Field
  name="age"
  validators={{
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value < 13 ? 'You must be 13 to make an account' : undefined
    },
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

同步和非同步驗證可以共存。例如，可以在同一欄位上同時定義 `onBlur` 和 `onBlurAsync`：

```svelte
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
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

同步驗證方法（`onBlur`）會先執行，非同步方法（`onBlurAsync`）只有在同步方法（`onBlur`）成功時才會執行。若要改變此行為，請將 `asyncAlways` 選項設為 `true`，這樣無論同步方法的結果如何，非同步方法都會執行。

### 內建防抖動 (Debouncing)

雖然非同步呼叫是針對資料庫進行驗證的好方法，但在每次按鍵時發送網路請求可能會對您的資料庫造成 DDoS 攻擊。

相反地，我們提供了一個簡單的方法來防抖動您的 `async` 呼叫，只需添加一個屬性：

```svelte
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsync: async ({ value }) => {
      // ...
    },
  }}
>
  <!-- ... -->
</form.Field>
```

這將以 500 毫秒的延遲防抖動每個非同步呼叫。您甚至可以針對每個驗證屬性覆寫此設定：

```svelte
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
>
  <!-- ... -->
</form.Field>
```

> 這將每 1500 毫秒執行一次 `onChangeAsync`，而 `onBlurAsync` 將每 500 毫秒執行一次。

## 透過結構描述 (Schema) 函式庫進行驗證

雖然函式提供了更多靈活性和自訂驗證的方式，但它們可能有些冗長。為了解決這個問題，有一些函式庫提供了基於結構描述的驗證，使簡寫和型別嚴格的驗證變得更加容易。您也可以為整個表單定義一個結構描述並將其傳遞到表單層級，錯誤會自動傳播到欄位。

### 標準結構描述函式庫

TanStack Form 原生支援所有遵循 [Standard Schema 規範](https://github.com/standard-schema/standard-schema) 的函式庫，最著名的包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_注意：_ 請確保使用結構描述函式庫的最新版本，因為舊版本可能尚未支援 Standard Schema。

要使用這些函式庫的結構描述，您可以像使用自訂函式一樣將其傳遞給 `validators` props：

```svelte
<script>
  import { z } from 'zod'

  // ...

  const form = createForm(() => ({
    // ...
  }))
</script>

<form.Field
  name="age"
  validators={{
    onChange: z.number().gte(13, 'You must be 13 to make an account'),
  }}
>
  <!-- ... -->
</form.Field>
```

表單和欄位層級的非同步驗證也支援：

```svelte
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
>
  <!-- ... -->
</form.Field>
```

## 防止提交無效表單

`onChange`、`onBlur` 等回調也會在提交表單時執行，如果表單無效，提交會被阻止。

表單狀態物件有一個 `canSubmit` 標誌，當任何欄位無效且表單已被觸碰時，該標誌為 false（`canSubmit` 在表單被觸碰之前為 true，即使某些欄位根據其 `onChange`/`onBlur` props「技術上」無效）。

您可以透過 `form.Subscribe` 訂閱它，並使用該值來實現例如在表單無效時禁用提交按鈕（實際上，禁用按鈕不可訪問，請改用 `aria-disabled`）。

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    /* ... */
  }))
</script>

<!-- ... -->

<!-- 動態提交按鈕 -->
<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
  children={(state) => (
    <button type="submit" disabled={!state().canSubmit}>
      {state().isSubmitting ? '...' : 'Submit'}
    </button>
  )}
>
  {#snippet children(state)}
    <button type="submit" disabled={!state.canSubmit}>
      {state.isSubmitting ? '...' : 'Submit'}
    </button>
  {/snippet}
</form.Subscribe>
```
