---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-25T20:33:34.857Z'
id: linked-fields
title: 關聯欄位
---
## 連結兩個表單欄位

您可能會遇到需要將兩個欄位連結在一起的情況：當一個欄位驗證失敗時，另一個欄位的值已發生變化。  
其中一種常見情境是同時擁有 `password` 和 `confirm_password` 欄位時，  
您會希望當 `password` 的值不匹配時，`confirm_password` 欄位顯示錯誤；  
無論是哪個欄位觸發了值變更。

想像以下使用者流程：

- 使用者更新確認密碼欄位
- 使用者更新非確認密碼欄位

在這個範例中，表單仍會顯示錯誤，  
因為「確認密碼」欄位的驗證並未重新執行以標記為已接受。

為了解決這個問題，我們需要確保在密碼欄位更新時重新執行「確認密碼」的驗證。  
您可以透過在 `confirm_password` 欄位添加 `onChangeListenTo` 屬性來實現。

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

同樣地，此機制也適用於 `onBlurListenTo` 屬性，當欄位失去焦點時會重新執行驗證。
