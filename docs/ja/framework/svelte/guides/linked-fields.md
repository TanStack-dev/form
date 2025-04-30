---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:27:59.954Z'
id: linked-fields
title: 連動フィールド
---

## 2つのフォームフィールドを連携させる

あるフィールドの値が変更されたときに、別のフィールドのバリデーションを実行したい場合があります。
例えば、`password`フィールドと`confirm_password`フィールドがあり、
`password`の値と一致しない場合に`confirm_password`フィールドでエラーを表示したい場合です。
どちらのフィールドで値が変更されても、この動作を実現する必要があります。

以下のユーザーフローを想像してください:

- ユーザーが確認用パスワードフィールドを更新
- ユーザーが通常のパスワードフィールドを更新

この例では、フォームには依然としてエラーが表示されたままになります。
なぜなら「確認用パスワード」フィールドのバリデーションが再実行されず、受け入れられたとマークされていないからです。

この問題を解決するには、パスワードフィールドが更新されたときに「確認用パスワード」のバリデーションを再実行する必要があります。
これを行うには、`confirm_password`フィールドに`onChangeListenTo`プロパティを追加します。

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      password: '',
      confirm_password: '',
    },
    // ...
  }))
</script>

<div>
  <form.Field name="password">
    {#snippet children(field)}
      <label>
        <div>Password</div>
        <input
          value={field.state.value}
          onChange={(e) => field.handleChange(e.target.value)}
        />
      </label>
    {/snippet}
  </form.Field>
  <form.Field
    name="confirm_password"
    validators={{
      onChangeListenTo: ['password'],
      onChange: ({ value, fieldApi }) => {
        if (value !== fieldApi.form.getFieldValue('password')) {
          return 'Passwords do not match'
        }
        return undefined
      },
    }}
  >
    {#snippet children(field)}
      <div>
        <label>
          <div>Confirm Password</div>
          <input
            value={field.state.value}
            onchange={(e) => field.handleChange(e.target.value)}
          />
        </label>
        {#each field.state.meta.errors as err}
          <div>{err}</div>
        {/each}
      </div>
    {/snippet}
  </form.Field>
</div>
```

同様の動作は`onBlurListenTo`プロパティでも実現でき、フィールドがブラー（フォーカスを失った）時にバリデーションが再実行されます。
