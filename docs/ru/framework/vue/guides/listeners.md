---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-05-06T20:27:19.057Z'
id: listeners
title: Side effects for event triggers
---

Для ситуаций, когда необходимо "воздействовать" или "реагировать" на триггеры, существует API слушателей (listener API). Например, если разработчик хочет сбросить значение поля формы в результате изменения другого поля, следует использовать API слушателей.

Представьте следующий пользовательский сценарий:

- Пользователь выбирает страну из выпадающего списка.
- Затем пользователь выбирает регион из другого выпадающего списка.
- Пользователь изменяет выбранную страну на другую.

В этом примере при изменении страны выбранный регион необходимо сбросить, так как он больше не действителен. С помощью API слушателей можно подписаться на событие `onChange` и сбросить поле "province" при срабатывании слушателя.

События, на которые можно подписаться:

- `onChange`
- `onBlur`
- `onMount`
- `onSubmit`

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
