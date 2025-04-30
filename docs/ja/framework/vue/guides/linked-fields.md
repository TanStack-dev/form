---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:26:08.231Z'
id: linked-fields
title: 連動フィールド
---

## 2つのフォームフィールドを連携させる

あるフィールドの値が変更されたときに、別のフィールドのバリデーションを行いたい場合があります。
例えば、`password`フィールドと`confirm_password`フィールドがある場合、
`password`の値と一致しないときに`confirm_password`フィールドでエラーを表示したい場合です。
どちらのフィールドが値の変更をトリガーしたかに関係なく動作させます。

以下のユーザーフローを想像してください:

- ユーザーが確認用パスワードフィールドを更新
- ユーザーが通常のパスワードフィールドを更新

この例では、フォームにはまだエラーが表示されたままです。
「確認用パスワード」フィールドのバリデーションが再実行されていないためです。

この問題を解決するには、パスワードフィールドが更新されたときに「確認用パスワード」のバリデーションを再実行する必要があります。
これを行うには、`confirm_password`フィールドに`onChangeListenTo`プロパティを追加します。

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

同様の動作は`onBlurListenTo`プロパティでも実現でき、フィールドがブラー（フォーカスを失った）時にバリデーションを再実行します。
