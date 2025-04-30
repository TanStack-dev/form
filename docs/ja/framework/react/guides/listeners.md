---
source-updated-at: '2025-04-27T15:54:55.000Z'
translation-updated-at: '2025-04-30T21:25:22.806Z'
id: listeners
title: リスナー
---

## イベントトリガーにおける副作用 (Side effects)

トリガーに対して「影響を与える」または「反応する」必要がある場合、リスナーAPIを使用できます。例えば、開発者が他のフィールドの変更に応じてフォームフィールドをリセットしたい場合、リスナーAPIを利用します。

以下のユーザーフローを想定してください:

- ユーザーがドロップダウンから国を選択
- ユーザーが別のドロップダウンから州を選択
- ユーザーが選択した国を変更

この例では、ユーザーが国を変更した際、選択された州は無効になるためリセットする必要があります。リスナーAPIを使用すると、`onChange`イベントを購読し、リスナーが発火した時に「province」フィールドにリセットをディスパッチできます。

「リッスン」可能なイベントは以下の通りです:

- `onChange`
- `onBlur`
- `onMount`
- `onSubmit`

```tsx
function App() {
  const form = useForm({
    defaultValues: {
      country: '',
      province: '',
    },
    // ...
  })

  return (
    <div>
      <form.Field
        name="country"
        listeners={{
          onChange: ({ value }) => {
            console.log(`Country changed to: ${value}, resetting province`)
            form.setFieldValue('province', '')
          },
        }}
      >
        {(field) => (
          <label>
            <div>Country</div>
            <input
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
          </label>
        )}
      </form.Field>

      <form.Field name="province">
        {(field) => (
          <label>
            <div>Province</div>
            <input
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
          </label>
        )}
      </form.Field>
    </div>
  )
}
```

### 組み込みのデバウンス (Debouncing)

リスナー内でAPIリクエストを行う場合、パフォーマンス問題を引き起こす可能性があるため、呼び出しをデバウンスしたいことがあります。
`onChangeDebounceMs`または`onBlurDebounceMs`を追加するだけで、リスナーを簡単にデバウンスできます。

```tsx
<form.Field
  name="country"
  listeners={{
    onChangeDebounceMs: 500, // 500msデバウンス
    onChange: ({ value }) => {
      console.log(`Country changed to: ${value} without a change within 500ms, resetting province`)
      form.setFieldValue('province', '')
    },
  }}
>
  {(field) => (
    /* ... */
  )}
</form.Field>
```

### フォームレベルのリスナー

より高いレベルでは、リスナーはフォームレベルでも利用可能で、`onMount`と`onSubmit`イベントへのアクセスを提供し、`onChange`と`onBlur`をフォームのすべての子要素に伝播させます。フォームレベルのリスナーも前述と同じ方法でデバウンスできます。

`onMount`と`onSubmit`リスナーは以下のプロパティにアクセスできます:

- `formApi`

`onChange`と`onBlur`リスナーは以下にアクセス可能です:

- `fieldApi`
- `formApi`

```tsx
const form = useForm({
  listeners: {
    onMount: ({ formApi }) => {
      // カスタムロギングサービス
      loggingService('mount', formApi.state.values)
    },

    onChange: ({ formApi, fieldApi }) => {
      // 自動保存ロジック
      if (formApi.state.isValid) {
        formApi.handleSubmit()
      }

      // fieldApiはイベントをトリガーしたフィールドを表します
      console.log(fieldApi.name, fieldApi.state.value)
    },
    onChangeDebounceMs: 500,
  },
})
```
