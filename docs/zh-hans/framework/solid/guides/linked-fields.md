---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-12T04:11:07.786Z'
id: linked-fields
title: 关联字段
---
## 联动两个表单字段

在某些场景下，您可能需要将两个表单字段关联起来：当其中一个字段的值发生变化时，触发另一个字段的验证逻辑。  
典型用例是当您同时拥有 `password` 和 `confirm_password` 字段时，  
需要确保 `confirm_password` 在值与 `password` 不匹配时显示错误——  
无论哪个字段触发了值变更。

假设以下用户操作流程：

- 用户更新确认密码字段
- 用户更新原始密码字段

在此示例中，表单仍会显示错误状态，  
因为 "确认密码" 字段的验证逻辑未被重新执行以标记为通过状态。

解决方案是确保密码字段更新时重新执行 "确认密码" 的验证逻辑。  
为此，您可以在 `confirm_password` 字段上添加 `onChangeListenTo` 属性。

```tsx
export default function App() {
  const form = createForm(() => ({
    defaultValues: {
      password: '',
      confirm_password: '',
    },
    // ...
  }))

  return (
    <div>
      <form.Field name="password">
        {(field) => (
          <label>
            <div>Password</div>
            <input
              value={field().state.value}
              onChange={(e) => field().handleChange(e.target.value)}
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
                value={field().state.value}
                onChange={(e) => field().handleChange(e.target.value)}
              />
            </label>
            <Index each={field().state.meta.errors}>
              {(err) => <div>{err()}</div>}
            </Index>
          </div>
        )}
      </form.Field>
    </div>
  )
}
```

该机制同样适用于 `onBlurListenTo` 属性，当字段失去焦点时会重新执行验证逻辑。
