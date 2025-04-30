---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-04-30T21:25:19.914Z'
id: submission-handling
title: 送信処理
---

## 送信処理に追加データを渡す

複数の種類の送信動作（例えば別のページに戻るか、フォームに留まるかなど）が必要な場合があります。
これは`onSubmitMeta`プロパティを指定することで実現できます。このメタデータは`onSubmit`関数に渡されます。

> 注: `form.handleSubmit()`がメタデータなしで呼び出された場合、指定されたデフォルト値が使用されます。

```tsx
import { useForm } from '@tanstack/react-form'

type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null
}

// form.handleSubmit()を呼び出す際にメタデータは必須ではありません。
// メタデータが渡されなかった場合に使用するデフォルト値を指定します
const defaultMeta: FormMeta = {
  submitAction: null,
}

function App() {
  const form = useForm({
    defaultValues: {
      data: '',
    },
    // 送信時に期待するメタ値を定義
    onSubmitMeta: defaultMeta,
    onSubmit: async ({ value, meta }) => {
      // handleSubmit経由で渡された値を使用して処理を実行
      console.log(`Selected action - ${meta.submitAction}`, value)
    },
  })

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault()
        e.stopPropagation()
      }}
    >
      {/* ... */}
      <button
        type="submit"
        // onSubmitMetaで指定されたデフォルト値を上書き
        onClick={() => form.handleSubmit({ submitAction: 'continue' })}
      >
        送信して続ける
      </button>
      <button
        type="submit"
        onClick={() => form.handleSubmit({ submitAction: 'backToMenu' })}
      >
        送信してメニューに戻る
      </button>
    </form>
  )
}
```

## スタンダードスキーマでデータを変換する

Tanstack Formはバリデーションのために[スタンダードスキーマサポート](./validation.md)を提供していますが、スキーマの出力データは保持しません。

`onSubmit`関数に渡される値は常に入力データです。スタンダードスキーマの出力データを受け取るには、`onSubmit`関数内でパースします:

```tsx
const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Formはスタンダードスキーマの入力型を使用します
const defaultValues: z.input<typeof schema> = {
  age: '13',
}

const form = useForm({
  defaultValues,
  validators: {
    onChange: schema,
  },
  onSubmit: ({ value }) => {
    const inputAge: string = value.age
    // 変換された値を取得するためにスキーマを通します
    const result = schema.parse(value)
    const outputAge: number = result.age
  },
})
```
