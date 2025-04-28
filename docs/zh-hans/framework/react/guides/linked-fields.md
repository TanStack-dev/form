---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-12T04:08:50.083Z'
id: linked-fields
title: 关联字段
---

## 联动两个表单字段

在某些场景下，您可能需要将两个字段进行联动：当其中一个字段的值发生变化时，另一个字段需要重新验证。一个典型用例是同时存在 `password` 和 `confirm_password` 字段的情况，此时需要确保当 `password` 字段值不匹配时，`confirm_password` 字段能立即显示错误——无论哪个字段触发了值的变化。

设想以下用户操作流程：

- 用户更新确认密码字段
- 用户更新原始密码字段

在此示例中，表单仍会显示错误状态，因为"确认密码"字段的验证逻辑没有重新执行以标记为通过状态。

为解决这个问题，我们需要确保当密码字段更新时，"确认密码"字段的验证会重新执行。可以通过给 `confirm_password` 字段添加 `onChangeListenTo` 属性来实现：

```tsx
function App() {
  const form = useForm({
    defaultValues: {
      password: '',
      confirm_password: '',
    },
    // ...
  })

  return (
    <div>
      <form.Field name="password">
        {(field) => (
          <label>
            <div>Password</div>
            <input
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
          </label>
        )}
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
        {(field) => (
          <div>
            <label>
              <div>Confirm Password</div>
              <input
                value={field.state.value}
                onChange={(e) => field.handleChange(e.target.value)}
              />
            </label>
            {field.state.meta.errors.map((err) => (
              <div key={err}>{err}</div>
            ))}
          </div>
        )}
      </form.Field>
    </div>
  )
}
```

同样的机制也适用于 `onBlurListenTo` 属性，该属性会在字段失去焦点时重新执行验证逻辑。
