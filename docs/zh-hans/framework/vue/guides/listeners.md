---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-05-06T20:17:05.937Z'
id: listeners
title: Side effects for event triggers
---

## 事件触发器的副作用处理

当您需要“影响”或“响应”某些触发行为时，可以使用监听器 (listener) API。例如，如果您作为开发者希望在某个字段变化时重置另一个表单字段，就可以使用监听器 API 来实现。

设想以下用户操作流程：

- 用户从下拉菜单中选择一个国家
- 用户接着从另一个下拉菜单中选择一个省份
- 用户将已选国家更改为其他选项

在这个例子中，当用户更改国家时，已选的省份需要被重置，因为它不再有效。通过监听器 API，我们可以订阅 onChange 事件，并在监听器触发时对“省份”字段执行重置操作。

可被“监听”的事件包括：

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
