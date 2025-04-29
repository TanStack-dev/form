---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-29T23:30:21.643Z'
id: form-validation
title: 表單驗證
---

TanStack Form 的核心功能之一是驗證 (validation) 概念。TanStack Form 讓驗證變得高度可自訂：

- 您可以控制何時執行驗證（變更時、輸入時、失焦時、提交時...）
- 驗證規則可以在欄位 (field) 層級或表單 (form) 層級定義
- 驗證可以是同步或非同步的（例如 API 呼叫的結果）

## 驗證何時執行？

由您決定！`<Field />` 元件接受一些回調函式作為 props，例如 `onChange` 或 `onBlur`。這些回調會接收欄位的當前值以及 `fieldAPI` 物件，以便您執行驗證。如果發現驗證錯誤，只需將錯誤訊息作為字串返回，它就會出現在 `field.state.meta.errors` 中。

以下是一個範例：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

在上面的範例中，驗證在每次按鍵時執行（`onChange`）。如果我們希望驗證在欄位失焦時執行，可以這樣修改程式碼：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onBlur: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <!-- 我們始終需要實作 onChange，以便 TanStack Form 接收變更 -->
      <!-- 監聽欄位的 onBlur 事件 -->
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

因此，您可以通過實作所需的回調來控制驗證的執行時機。您甚至可以在不同時間執行不同的驗證邏輯：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
      onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <!-- 我們始終需要實作 onChange，以便 TanStack Form 接收變更 -->
      <!-- 監聽欄位的 onBlur 事件 -->
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

在上面的範例中，我們在同一欄位的不同時間（每次按鍵和欄位失焦時）驗證不同的內容。由於 `field.state.meta.errors` 是一個陣列，所有相關的錯誤都會在特定時間顯示。您也可以使用 `field.state.meta.errorMap` 根據驗證執行的時機（onChange、onBlur 等）獲取錯誤。更多關於顯示錯誤的資訊如下。

## 顯示錯誤

一旦設置好驗證，您可以將錯誤從陣列映射到 UI 中顯示：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

或者使用 `errorMap` 屬性來存取特定的錯誤：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
      <em role="alert" v-if="field.state.meta.errorMap['onChange']">{{
        field.state.meta.errorMap['onChange']
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

值得一提的是，我們的 `errors` 陣列和 `errorMap` 與驗證器返回的類型相匹配。這意味著：

```vue
<form.Field
  name="age"
  :validators="{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }"
>
  <template v-slot="{ field }">
      <!-- ... -->
      <!-- errorMap.onChange 的類型是 `{isOldEnough: false} | undefined` -->
	  <!-- meta.errors 的類型是 `Array<{isOldEnough: false} | undefined>` -->
      <em v-if="!field.state.meta.errorMap['onChange']?.isOldEnough">The user is not old enough</em>
  </template>
</form.Field>
```

## 欄位層級與表單層級的驗證

如上所示，每個 `<Field>` 通過 `onChange`、`onBlur` 等回調接受自己的驗證規則。也可以通過將類似的回調傳遞給 `useForm()` 函數，在表單層級（而不是逐個欄位）定義驗證規則。

範例：

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

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
const formErrorMap = form.useStore((state) => state.errorMap)
</script>

<template>
  <!-- ... -->
  <div v-if="formErrorMap.onChange">
    <em role="alert">
      There was an error on the form: {{ formErrorMap.onChange }}
    </em>
  </div>
  <!-- ... -->
</template>
```

### 從表單驗證器設置欄位層級錯誤

您可以從表單的驗證器中設置欄位的錯誤。一個常見的用例是在表單的 `onSubmitAsync` 驗證器中調用單個 API 端點來驗證所有欄位。

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    age: 0,
    socials: [],
    details: {
      email: '',
    },
  },
  validators: {
    // 以與欄位相同的方式向表單添加驗證器
    onSubmitAsync({ value }) {
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
</script>

<template>
  <!-- ... -->
</template>
```

> 值得一提的是，如果表單驗證函數返回錯誤，該錯誤可能會被欄位特定的驗證覆蓋。
>
> 這意味著：
>
> ```vue
> <script setup lang="ts">
> import { useForm } from '@tanstack/vue-form'
>
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
> </script>
>
> <template>
>   <!-- ... -->
>   <form.Field
>     name="age"
>     :validators="{
>       onChange: ({ value }) => (value % 2 === 0 ? 'Must be odd!' : undefined),
>     }"
>   >
>     <template v-slot="{ field }">
>       <!-- ... -->
>     </template>
>   </form.Field>
> </template>
> ```
>
> 即使表單層級驗證返回了 'Too young!' 錯誤，也只會顯示 `'Must be odd!'`。

## 非同步函數驗證

雖然我們認為大多數驗證是同步的，但在許多情況下，網絡呼叫或其他非同步操作對驗證很有用。

為此，我們提供了專用的 `onChangeAsync`、`onBlurAsync` 等方法：

```vue
<script setup lang="ts">
// ...

const onChangeAge = async ({ value }) => {
  await new Promise((resolve) => setTimeout(resolve, 1000))
  return value < 13 ? 'You must be 13 to make an account' : undefined
}
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChangeAsync: onChangeAge,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @input="
          (e) =>
            field.handleChange((e.target as HTMLInputElement).valueAsNumber)
        "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

同步和非同步驗證可以共存。例如，可以在同一欄位上同時定義 `onBlur` 和 `onBlurAsync`：

```vue
<script setup lang="ts">
// ...

const onBlurAge = ({ value }) => (value < 0 ? 'Invalid value' : undefined)

const onBlurAgeAsync = async ({ value }) => {
  const currentAge = await fetchCurrentAgeOnProfile()
  return value < currentAge ? 'You can only increase the age' : undefined
}
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onBlur: onBlurAge,
      onBlurAsync: onBlurAgeAsync,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="
          (e) =>
            field.handleChange((e.target as HTMLInputElement).valueAsNumber)
        "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

同步驗證方法（`onBlur`）首先運行，只有在同步方法（`onBlur`）成功時才會運行非同步方法（`onBlurAsync`）。要改變這種行為，將 `asyncAlways` 選項設置為 `true`，非同步方法將無論同步方法的結果如何都會運行。

### 內建防抖 (Debouncing)

雖然非同步呼叫是驗證數據庫的好方法，但在每次按鍵時運行網絡請求是對數據庫進行 DDOS 攻擊的好方法。

相反，我們通過添加一個屬性來提供一種簡單的方法來防抖您的 `async` 呼叫：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :async-debounce-ms="500"
    :validators="{
      onChangeAsync: async ({ value }) => {
        // ...
      },
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

這將以 500 毫秒的延遲防抖每個非同步呼叫。您甚至可以針對每個驗證屬性覆蓋此設置：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :async-debounce-ms="500"
    :validators="{
      onChangeAsyncDebounceMs: 1500,
      onChangeAsync: async ({ value }) => {
        // ...
      },
      onBlurAsync: async ({ value }) => {
        // ...
      },
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

這將每 1500 毫秒運行一次 `onChangeAsync`，而 `onBlurAsync` 將每 500 毫秒運行一次。

## 通過 Schema 函式庫進行驗證

雖然函數提供了更多靈活性和自訂驗證的能力，但它們可能有點冗長。為了解決這個問題，有一些函式庫提供了基於 schema 的驗證，使簡寫和類型嚴格的驗證變得更加容易。您還可以為整個表單定義一個 schema 並將其傳遞到表單層級，錯誤將自動傳播到欄位。

### 標準 Schema 函式庫

TanStack Form 原生支持所有遵循 [Standard Schema 規範](https://github.com/standard-schema/standard-schema) 的函式庫，最著名的是：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_注意：_ 確保使用 schema 函式庫的最新版本，因為舊版本可能還不支持 Standard Schema。

> 驗證不會提供轉換後的值。有關更多信息，請參閱 [提交處理](./submission-handling.md)。

要使用這些函式庫的 schema，您可以像使用自訂函數一樣將它們傳遞給 `validators` props：

```vue
<script setup lang="ts">
import { z } from 'zod'
// ...

const form = useForm({
  // ...
})
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: z.number().gte(13, 'You must be 13 to make an account'),
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

表單和欄位層級的非同步驗證也受支持：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
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
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

如果您需要對 Standard Schema 驗證進行更多控制，可以像這樣將 Standard Schema 與回調函數結合：

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value, fieldApi }) => {
        const errors = fieldApi.parseValueWithSchema(
          z.number().gte(13, 'You must be 13 to make an account'),
        )

        if (errors) return errors

        // 繼續您的驗證
      },
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## 防止提交無效表單

`onChange`、`onBlur` 等回調也會在提交表單時運行，如果表單無效，提交將被阻止。

表單狀態物件有一個 `canSubmit` 標誌，當任何欄位無效且表單已被觸碰時為 false（`canSubmit` 在表單被觸碰之前為 true，即使某些
