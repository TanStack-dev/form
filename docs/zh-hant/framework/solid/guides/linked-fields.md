---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-25T20:47:37.261Z'
id: linked-fields
title: 關聯欄位
---

你可能會遇到需要將兩個表單欄位連結在一起的情況：當其中一個欄位需要根據另一個欄位的值變更進行驗證時。
一個常見的使用情境是當你同時有 `password` 和 `confirm_password` 欄位時，
你會希望當 `password` 的值不匹配時，`confirm_password` 能顯示錯誤；
無論是哪個欄位觸發了值變更。

想像以下使用者流程：

- 使用者更新確認密碼欄位。
- 使用者更新非確認密碼欄位。

在這個例子中，表單仍會顯示錯誤，
因為「確認密碼」欄位的驗證尚未重新執行以標記為已接受。

為了解決這個問題，我們需要確保當密碼欄位更新時，「確認密碼」的驗證會重新執行。
要做到這一點，你可以在 `confirm_password` 欄位中加入 `onChangeListenTo` 屬性。

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

同樣地，這也適用於 `onBlurListenTo` 屬性，該屬性會在欄位失去焦點時重新執行驗證。
