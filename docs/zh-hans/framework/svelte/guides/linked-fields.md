---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-29T23:14:55.021Z'
id: linked-fields
title: 联动字段
---

## 联动两个表单字段

在某些场景下，您可能需要将两个字段进行联动验证：当其中一个字段的值发生变化时，另一个字段需要重新验证。典型的应用场景是同时存在 `password` 和 `confirm_password` 字段时，无论哪个字段触发了值的变化，都需要确保 `confirm_password` 在值与 `password` 不匹配时显示错误。

考虑以下用户操作流程：

- 用户修改确认密码字段
- 用户修改原始密码字段

在这个例子中，表单仍会显示错误状态，因为"确认密码"字段的验证逻辑没有重新执行以更新验证状态。

解决方案是确保在密码字段更新时重新执行"确认密码"的验证逻辑。为此，您可以在 `confirm_password` 字段上添加 `onChangeListenTo` 属性。

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
        <div>密码</div>
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
          return '密码不匹配'
        }
        return undefined
      },
    }}
  >
    {#snippet children(field)}
      <div>
        <label>
          <div>确认密码</div>
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

同样的机制也适用于 `onBlurListenTo` 属性，该属性会在字段失去焦点时重新执行验证逻辑。
