---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-25T20:40:57.676Z'
id: basic-concepts
title: 基本概念
---

本頁介紹 `@tanstack/vue-form` 函式庫中使用的基本概念與術語。熟悉這些概念將幫助您更有效地理解與運用此函式庫。

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

表單實例是一個代表單一表單的物件，提供操作表單的方法與屬性。您可以使用 `useForm` 函式建立表單實例。該函式接受包含 `onSubmit` 函式的物件，此函式會在表單提交時被呼叫。

```js
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // 處理表單資料
    console.log(value)
  },
})
```

您也可以不使用 `formOptions`，直接透過獨立的 `useForm` API 建立表單實例：

```ts
const form = useForm({
  onSubmit: async ({ value }) => {
    // 處理表單資料
    console.log(value)
  },
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## 欄位 (Field)

欄位代表單一表單輸入元素，例如文字輸入框或核取方塊。欄位透過表單實例提供的 `form.Field` 元件建立。該元件接受 `name` 屬性，需與表單預設值中的鍵名匹配。同時也接受透過 `v-slot` 指令定義的作用域插槽，該插槽會接收一個 `field` 物件作為參數。

範例：

```vue
<template>
  <!-- ... -->
  <form.Field name="fullName">
    <template v-slot="{ field }">
      <input
        :name="field.name"
        :value="field.state.value"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange(e.target.value)"
      />
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## 欄位狀態 (Field State)

每個欄位都有自身的狀態，包含當前值、驗證狀態、錯誤訊息與其他元資料。您可以透過 `field.state` 屬性存取欄位狀態。

範例：

```js
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

有三種欄位狀態能有效反映使用者互動行為：當使用者點擊/切換至欄位時標記為 _"touched"_、使用者未改變值時為 _"pristine"_、值被修改後則標記為 _"dirty"_。可透過以下標誌檢查這些狀態：

```js
const { isTouched, isPristine, isDirty } = field.state.meta
```

![欄位狀態](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## 欄位 API (Field API)

欄位 API 是由 `v-slot` 指令提供的作用域插槽物件。該插槽接收名為 `field` 的參數，提供操作欄位狀態的方法與屬性。

範例：

```vue
<template v-slot="{ field }">
  <input
    :name="field.name"
    :value="field.state.value"
    @blur="field.handleBlur"
    @input="(e) => field.handleChange(e.target.value)"
  />
</template>
```

## 驗證 (Validation)

`@tanstack/vue-form` 原生支援同步與非同步驗證。驗證函式可透過 `validators` 屬性傳遞給 `form.Field` 元件。

範例：

```vue
<template>
    <!-- ... -->
    <form.Field
        name="firstName"
        :validators="{
            onChange: ({ value }) =>
                !value
                    ? `請輸入名字`
                    : value.length < 3
                        ? `名字至少需 3 個字元`
                        : undefined,
            onChangeAsync: async ({ value }) => {
                await new Promise((resolve) => setTimeout(resolve, 1000))
                return value.includes('error') && '名字中不得包含 "error"'
        },
    }"
    >
        <template v-slot="{ field }">
            <input
                :value="field.state.value"
                @input="(e) => field.handleChange(e.target.value)"
                @blur="field.handleBlur"
            />
            <FieldInfo :field="field" />
        </template>
    </form.Field>
    <!-- ... -->
</template>
```

## 標準結構描述驗證 (Validation with Standard Schema Libraries)

除了手動驗證選項外，我們也支援 [Standard Schema](https://github.com/standard-schema/standard-schema) 規範。

您可以使用任何實作此規範的函式庫定義結構描述，並將其傳遞給表單或欄位驗證器。

支援的函式庫包含：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'
import { z } from 'zod'

const form = useForm({
  // ...
})

const onChangeFirstName = z.string().refine(
  async (value) => {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return !value.includes('error')
  },
  {
    message: "名字中不得包含 'error'",
  },
)
</script>

<template>
  <!-- ... -->
  <form.Field
    name="firstName"
    :validators="{
      onChange: z.string().min(3, '名字至少需 3 個字元'),
      onChangeAsyncDebounceMs: 500,
      onChangeAsync: onChangeFirstName,
    }"
  >
    <template v-slot="{ field, state }">
      <label :htmlFor="field.name">名字：</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).value)"
        @blur="field.handleBlur"
      />
      <FieldInfo :state="state" />
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## 響應式 (Reactivity)

`@tanstack/vue-form` 提供多種訂閱表單與欄位狀態變更的方式，最主要是 `form.useStore` 方法與 `form.Subscribe` 元件。這些方法能讓您透過必要時才更新元件來優化表單渲染效能。

範例：

```vue
<script setup lang="ts">
// ...
const firstName = form.useStore((state) => state.values.firstName)
</script>

<template>
  <!-- ... -->
  <form.Subscribe>
    <template v-slot="{ canSubmit, isSubmitting }">
      <button type="submit" :disabled="!canSubmit">
        {{ isSubmitting ? '...' : '提交' }}
      </button>
    </template>
  </form.Subscribe>
  <!-- ... -->
</template>
```

## 監聽器 (Listeners)

`@tanstack/vue-form` 允許您對特定觸發事件做出反應，並「監聽」這些事件以執行副作用。

範例：

```vue
<template>
  <form.Field
    name="country"
    :listeners="{
      onChange: ({ value }) => {
        console.log(`國家變更為: ${value}, 重置省份欄位`)
        form.setFieldValue('province', '')
      },
    }"
  >
    <template v-slot="{ field }">
      <input
        :value="field.state.value"
        @input="(e) => field.handleChange(e.target.value)"
      />
    </template>
  </form.Field>
</template>
```

更多資訊請參閱 [監聽器](./listeners.md)

注意：不建議使用 `form.useField` 方法實現響應式，因該方法設計為需謹慎使用於 `form.Field` 元件內。建議改用 `form.useStore`。

## 陣列欄位 (Array Fields)

陣列欄位讓您能管理表單中的值列表，例如興趣清單。您可透過 `mode="array"` 屬性的 `form.Field` 元件建立陣列欄位。

操作陣列欄位時，可使用 `pushValue`、`removeValue`、`swapValues` 與 `moveValue` 方法來新增、移除與交換陣列中的值。

範例：

```vue
<template>
  <form @submit.prevent.stop="form.handleSubmit">
    <form.Field name="hobbies" mode="array">
      <template v-slot="{ field: hobbiesField }">
        <div>
          興趣
          <div>
            <div
              v-if="
                Array.isArray(hobbiesField.state.value) &&
                !hobbiesField.state.value.length
              "
            >
              尚未新增興趣。
            </div>
            <div v-else>
              <div v-for="(_, i) in hobbiesField.state.value" :key="i">
                <form.Field :name="`hobbies[${i}].name`">
                  <template v-slot="{ field }">
                    <div>
                      <label :for="field.name">名稱：</label>
                      <input
                        :id="field.name"
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="(e) => field.handleChange(e.target.value)"
                      />
                      <button
                        type="button"
                        @click="hobbiesField.removeValue(i)"
                      >
                        X
                      </button>
                      <FieldInfo :field="field" />
                    </div>
                  </template>
                </form.Field>
                <form.Field :name="`hobbies[${i}].description`">
                  <template v-slot="{ field }">
                    <div>
                      <label :for="field.name">描述：</label>
                      <input
                        :id="field.name"
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="(e) => field.handleChange(e.target.value)"
                      />
                      <FieldInfo :field="field" />
                    </div>
                  </template>
                </form.Field>
              </div>
            </div>
            <button
              type="button"
              @click="
                hobbiesField.pushValue({
                  name: '',
                  description: '',
                  yearsOfExperience: 0,
                })
              "
            >
              新增興趣
            </button>
          </div>
        </div>
      </template>
    </form.Field>
  </form>
</template>
```

以上是 `@tanstack/vue-form` 函式庫中使用的基本概念與術語。理解這些概念將幫助您更有效率地運用此函式庫，輕鬆建立複雜的表單。
