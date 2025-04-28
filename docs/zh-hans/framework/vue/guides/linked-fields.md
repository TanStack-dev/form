---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-12T04:09:57.125Z'
id: linked-fields
title: 关联字段
---

## 联动两个表单字段

在某些场景下，您可能需要将两个字段进行联动验证：当其中一个字段的值发生变化时，另一个字段需要重新验证。一个典型用例是同时存在 `password` 和 `confirm_password` 字段的情况，此时需要确保当 `password` 字段的值不匹配时，`confirm_password` 字段能立即显示错误——无论哪个字段触发了值的变化。

考虑以下用户操作流程：

- 用户更新确认密码字段
- 用户更新原始密码字段

在这个例子中，表单仍会显示错误状态，因为 "确认密码" 字段的验证逻辑没有重新执行以标记为验证通过。

为了解决这个问题，我们需要确保在密码字段更新时重新运行 "确认密码" 的验证逻辑。可以通过给 `confirm_password` 字段添加 `onChangeListenTo` 属性来实现：

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    password: '',
    confirm_password: '',
  },
  // ...
})
</script>

<template>
  <div>
    <form @submit.prevent.stop="form.handleSubmit">
      <div>
        <form.Field name="password">
          <template v-slot="{ field }">
            <div>密码:</div>
            <input
              :value="field.state.value"
              @input="
                (e) => field.handleChange((e.target as HTMLInputElement).value)
              "
            />
          </template>
        </form.Field>
        <form.Field
          name="confirm_password"
          :validators="{
            onChangeListenTo: ['password'],
            onChange: ({ value, fieldApi }) => {
              if (value !== fieldApi.form.getFieldValue('password')) {
                return '密码不匹配'
              }
              return undefined
            },
          }"
        >
          <template v-slot="{ field }">
            <div>确认密码:</div>
            <input
              :value="field.state.value"
              @input="
                (e) => field.handleChange((e.target as HTMLInputElement).value)
              "
            />
            <div v-for="(err, index) in field.state.meta.errors" :key="index">
              {{ err }}
            </div>
          </template>
        </form.Field>
      </div>
      <button type="submit">提交</button>
    </form>
  </div>
</template>
```

同样的机制也适用于 `onBlurListenTo` 属性，该属性会在字段失去焦点时重新运行验证逻辑。
