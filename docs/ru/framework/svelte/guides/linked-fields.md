---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T22:02:05.880Z'
id: linked-fields
title: Связанные поля
---

Вы можете столкнуться с необходимостью связать два поля формы, когда одно из них должно валидироваться при изменении значения другого поля.  
Один из примеров такого использования — поля `password` и `confirm_password`, где вы хотите, чтобы `confirm_password` выдавал ошибку, если его значение не совпадает с `password`, независимо от того, какое поле вызвало изменение.

Представьте следующий сценарий:

- Пользователь изменяет поле подтверждения пароля.
- Пользователь изменяет основное поле пароля.

В этом примере форма всё равно будет содержать ошибки, так как валидация поля "confirm password" не была перезапущена, чтобы отметить его как корректное.

Чтобы решить эту проблему, нужно убедиться, что валидация "confirm password" выполняется заново при обновлении поля пароля.  
Для этого можно добавить свойство `onChangeListenTo` в поле `confirm_password`.

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

Аналогично работает свойство `onBlurListenTo`, которое перезапускает валидацию при потере фокуса полем.
