---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-05-06T20:19:23.572Z'
id: listeners
title: Side effects for event triggers
---

トリガーに対して「影響を与える」または「反応する」必要がある場合、リスナーAPI (listener API) を使用できます。例えば、開発者が他のフィールドの変更に応じてフォームフィールドをリセットしたい場合、リスナーAPIを利用します。

以下のユーザーフローを想像してください:

- ユーザーがドロップダウンから国を選択
- ユーザーが別のドロップダウンから州/省を選択
- ユーザーが選択した国を変更

この例では、ユーザーが国を変更した際、選択された州/省は無効になるためリセットする必要があります。リスナーAPIを使用すると、`onChange`イベントを購読し、リスナーが発火した時に「province」フィールドをリセットできます。

「監視」可能なイベントは以下の通りです:

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
