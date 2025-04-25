---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-25T20:44:46.365Z'
id: linked-fields
title: 關聯欄位
---
## 連結兩個表單欄位

您可能會遇到需要將兩個欄位連結在一起的情況：當其中一個欄位需要根據另一個欄位的值變更進行驗證時。  
一個常見的使用情境是當您同時擁有 `password` 和 `confirm_password` 欄位時，  
您希望當 `confirm_password` 的值與 `password` 不符時顯示錯誤，  
無論是哪個欄位觸發了值變更。

想像以下使用者流程：

- 使用者更新確認密碼欄位
- 使用者更新原始密碼欄位

在此範例中，表單仍會顯示錯誤，  
因為「確認密碼」欄位的驗證並未重新執行以標記為通過狀態。

為了解決這個問題，我們需要確保在密碼欄位更新時重新執行「確認密碼」的驗證。  
您可以透過在 `confirm_password` 欄位添加 `onChangeListenTo` 屬性來實現：

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
            <div>密碼：</div>
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
                return '密碼不相符'
              }
              return undefined
            },
          }"
        >
          <template v-slot="{ field }">
            <div>確認密碼：</div>
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

同樣地，此機制也適用於 `onBlurListenTo` 屬性，該屬性會在欄位失去焦點時重新執行驗證。
