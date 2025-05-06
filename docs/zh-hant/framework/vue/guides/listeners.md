---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-05-06T20:18:20.656Z'
id: listeners
title: Side effects for event triggers
---

## 動機

在需要「影響」或「回應」觸發事件的情境下，可以使用監聽器 API (listener API)。舉例來說，當開發者需要在某個欄位變更時重置另一個表單欄位，就會使用監聽器 API。

想像以下使用者流程：

- 使用者從下拉選單中選擇國家
- 接著從另一個下拉選單中選擇省份
- 使用者將選定的國家變更為其他選項

在這個範例中，當使用者變更國家時，已選的省份需要被重置，因為它已不再有效。透過監聽器 API，我們可以訂閱 onChange 事件，並在監聽器觸發時對「province」欄位發送重置指令。

可被「監聽」的事件包括：

- onChange
- onBlur
- onMount
- onSubmit

```vue
<script setup>
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    country: '',
    province: '',
  },
  // ...
})
</script>

<template>
  <div>
    <form.Field
      name="country"
      :listeners="{
        onChange: ({ value }) => {
          console.log(`Country changed to: ${value}, resetting province`)
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

    <form.Field name="province">
      <template v-slot="{ field }">
        <input
          :value="field.state.value"
          @input="(e) => field.handleChange(e.target.value)"
        />
      </template>
    </form.Field>
  </div>
</template>
```
