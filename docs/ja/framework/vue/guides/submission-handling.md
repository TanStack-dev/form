---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:18:51.684Z'
id: submission-handling
title: Submission handling
---

## 送信処理に追加データを渡す

複数のタイプの送信動作（例えば別のページに戻る、またはフォームに留まるなど）が必要な場合があります。
これは`onSubmitMeta`プロパティを指定することで実現できます。このメタデータは`onSubmit`関数に渡されます。

> 注: `form.handleSubmit()`がメタデータなしで呼び出された場合、指定されたデフォルト値が使用されます。

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null
}

// form.handleSubmit()を呼び出す際にメタデータは必須ではありません。
// メタデータが渡されなかった場合に使用するデフォルト値を指定します
const defaultMeta: FormMeta = {
  submitAction: null,
}

const form = useForm({
  defaultValues: {
    data: '',
  },
  // 送信時に期待するメタ値を定義
  onSubmitMeta: defaultMeta,
  onSubmit: async ({ value, meta }) => {
    // handleSubmit経由で渡された値を使用して処理を実行
    console.log(`選択されたアクション - ${meta.submitAction}`, value)
  },
})
</script>

<template>
  <form @submit.prevent.stop="">
    <button
      type="submit"
      @click="
        () => {
          // onSubmitMetaで指定されたデフォルト値を上書き
          form.handleSubmit({ submitAction: 'continue' })
        }
      "
    >
      送信して続ける
    </button>
    <button
      type="submit"
      @click="() => form.handleSubmit({ submitAction: 'backToMenu' })"
    >
      送信してメニューに戻る
    </button>
  </form>
</template>
```

## スタンダードスキーマでデータを変換する

Tanstack Formはバリデーションのために[スタンダードスキーマサポート](./validation.md)を提供していますが、スキーマの出力データは保持しません。

`onSubmit`関数に渡される値は常に入力データです。スタンダードスキーマの出力データを受け取るには、`onSubmit`関数内でパースします:

```vue
<script setup lang="ts">
import { z } from 'zod'
import { useForm } from '@tanstack/vue-form'

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
    // 変換された値を取得するためにスキーマを通してパース
    const result = schema.parse(value)
    const outputAge: number = result.age
  },
})
</script>
<template>
  <!-- ... -->
</template>
```
