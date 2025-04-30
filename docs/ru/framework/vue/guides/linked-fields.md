---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T22:00:25.291Z'
id: linked-fields
title: Связанные поля
---

Вы можете столкнуться с необходимостью связать два поля формы, когда одно из них должно валидироваться при изменении значения другого поля.  
Один из таких случаев — наличие полей `password` и `confirm_password`, где вы хотите, чтобы `confirm_password` выдавал ошибку, если его значение не совпадает с `password`, независимо от того, какое поле вызвало изменение.

Представьте следующий сценарий:

- Пользователь изменяет поле подтверждения пароля.
- Пользователь изменяет основное поле пароля.

В этом примере форма всё равно будет содержать ошибки, так как валидация поля "подтверждения пароля" не была перезапущена, чтобы пометить его как корректное.

Чтобы решить эту проблему, нужно убедиться, что валидация "подтверждения пароля" перезапускается при обновлении поля пароля.  
Для этого можно добавить свойство `onChangeListenTo` к полю `confirm_password`.

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
            <div>Password:</div>
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
                return 'Passwords do not match'
              }
              return undefined
            },
          }"
        >
          <template v-slot="{ field }">
            <div>Confirm Password:</div>
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
      <button type="submit">Submit</button>
    </form>
  </div>
</template>
```

Аналогично работает свойство `onBlurListenTo`, которое перезапускает валидацию при потере фокуса полем.
