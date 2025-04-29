---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-25T20:49:18.805Z'
id: linked-fields
title: 關聯欄位
---

## 連結兩個表單欄位

您可能會遇到需要將兩個欄位連結在一起的情況：當其中一個欄位需要根據另一個欄位值的變動進行驗證時。
一個常見的使用情境是當您同時擁有 `password` 和 `confirm_password` 欄位時，
您會希望當 `confirm_password` 的值與 `password` 不符時顯示錯誤，
不論是哪個欄位觸發了值變更。

想像以下使用者流程：

- 使用者更新確認密碼欄位
- 使用者更新非確認密碼欄位

在這個範例中，表單仍會顯示錯誤，
因為「確認密碼」欄位的驗證尚未重新執行以標記為已接受。

為了解決這個問題，我們需要確保當密碼欄位更新時，「確認密碼」的驗證會被重新執行。
您可以透過在 `confirm_password` 欄位添加 `onChangeListenTo` 屬性來實現。

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

同樣地，這也適用於 `onBlurListenTo` 屬性，該屬性會在欄位失去焦點時重新執行驗證。
