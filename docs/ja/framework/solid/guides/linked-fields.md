---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:27:17.661Z'
id: linked-fields
title: 連動フィールド
---

## 2つのフォームフィールドを連携させる

あるフィールドの値が変更されたときに、別のフィールドのバリデーションを行いたい場合があります。
例えば、`password`フィールドと`confirm_password`フィールドがある場合、
`password`の値と一致しないときに`confirm_password`にエラーを表示したい場合です。
どちらのフィールドが変更をトリガーしたかに関わらず、この動作を実現します。

以下のユーザーフローを想像してください:

- ユーザーが確認用パスワードフィールドを更新
- ユーザーが通常のパスワードフィールドを更新

この例では、フォームには依然としてエラーが表示されたままになります。
なぜなら「確認用パスワード」フィールドのバリデーションが再実行されず、受け入れられたとマークされていないからです。

この問題を解決するには、パスワードフィールドが更新されたときに「確認用パスワード」のバリデーションを再実行する必要があります。
これを行うには、`confirm_password`フィールドに`onChangeListenTo`プロパティを追加します。

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

同様の動作は`onBlurListenTo`プロパティでも実現でき、フィールドがブラー（フォーカスを失った）時にバリデーションが再実行されます。
