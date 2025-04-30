---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:26:52.097Z'
id: basic-concepts
title: 基本概念
---

このページでは、`@tanstack/vue-form` ライブラリで使用される基本概念と用語を紹介します。これらの概念を理解することで、ライブラリをより効果的に活用できるようになります。

## フォームオプション

`formOptions` 関数を使用して、複数のフォーム間で共有可能なオプションを作成できます。

例:

```ts
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## フォームインスタンス

フォームインスタンスは個々のフォームを表すオブジェクトで、フォームを操作するためのメソッドとプロパティを提供します。`useForm` 関数を使用してフォームインスタンスを作成します。この関数は `onSubmit` 関数を含むオブジェクトを受け取り、フォームが送信されると呼び出されます。

```js
const form = useForm({
  ...formOpts,
  onSubmit: async ({ value }) => {
    // フォームデータで何か処理を行う
    console.log(value)
  },
})
```

`formOptions` を使用せずにスタンドアロンの `useForm` API でフォームインスタンスを作成することも可能です:

```ts
const form = useForm({
  onSubmit: async ({ value }) => {
    // フォームデータで何か処理を行う
    console.log(value)
  },
  defaultValues: {
    firstName: '',
    lastName: '',
    hobbies: [],
  } as Person,
})
```

## フィールド

フィールドは、テキスト入力やチェックボックスなどの単一のフォーム入力要素を表します。フィールドはフォームインスタンスが提供する `form.Field` コンポーネントを使用して作成します。このコンポーネントは、フォームのデフォルト値のキーと一致する必要がある `name` プロパティを受け取ります。また、`v-slot` ディレクティブで定義されたスコープ付きスロットも受け取り、`field` オブジェクトを引数として受け取ります。

例:

```vue
<template>
  <!-- ... -->
  <form.Field name="fullName">
    <template v-slot="{ field }">
      <input
        :name="field.name"
        :value="field.state.value"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange(e.target.value)"
      />
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## フィールドステート

各フィールドには、現在の値、検証ステータス、エラーメッセージ、その他のメタデータを含む独自のステートがあります。`field.state` プロパティを使用してフィールドのステートにアクセスできます。

例:

```js
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

ユーザーがフィールドとどのようにやり取りするかを確認するのに特に有用な3つのフィールドステートがあります。ユーザーがフィールドをクリック/タブすると「touched」、値が変更されるまで「pristine」、値が変更されると「dirty」とマークされます。以下のように `isTouched`、`isPristine`、`isDirty` フラグでこれらのステートを確認できます。

```js
const { isTouched, isPristine, isDirty } = field.state.meta
```

![フィールドステート](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## フィールドAPI

フィールドAPIは、`v-slot` ディレクティブを使用したスコープ付きスロットによって提供されるオブジェクトです。このスロットは `field` という名前の引数を受け取り、フィールドのステートを操作するためのメソッドとプロパティを提供します。

例:

```vue
<template v-slot="{ field }">
  <input
    :name="field.name"
    :value="field.state.value"
    @blur="field.handleBlur"
    @input="(e) => field.handleChange(e.target.value)"
  />
</template>
```

## バリデーション

`@tanstack/vue-form` は同期および非同期バリデーションを標準で提供します。バリデーション関数は `validators` プロパティを使用して `form.Field` コンポーネントに渡すことができます。

例:

```vue
<template>
    <!-- ... -->
    <form.Field
        name="firstName"
        :validators="{
            onChange: ({ value }) =>
                !value
                    ? `名を入力してください`
                    : value.length < 3
                        ? `名は3文字以上必要です`
                        : undefined,
            onChangeAsync: async ({ value }) => {
                await new Promise((resolve) => setTimeout(resolve, 1000))
                return value.includes('error') && '名に"error"を含めることはできません'
        },
    }"
    >
        <template v-slot="{ field }">
            <input
                :value="field.state.value"
                @input="(e) => field.handleChange(e.target.value)"
                @blur="field.handleBlur"
            />
            <FieldInfo :field="field" />
        </template>
    </form.Field>
    <!-- ... -->
</template>
```

## 標準スキーマライブラリを使用したバリデーション

手動で作成したバリデーションオプションに加えて、[Standard Schema](https://github.com/standard-schema/standard-schema) 仕様もサポートしています。

この仕様を実装した任意のライブラリを使用してスキーマを定義し、フォームまたはフィールドバリデータに渡すことができます。

サポートされているライブラリ:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'
import { z } from 'zod'

const form = useForm({
  // ...
})

const onChangeFirstName = z.string().refine(
  async (value) => {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return !value.includes('error')
  },
  {
    message: "名に'error'を含めることはできません",
  },
)
</script>

<template>
  <!-- ... -->
  <form.Field
    name="firstName"
    :validators="{
      onChange: z.string().min(3, '名は3文字以上必要です'),
      onChangeAsyncDebounceMs: 500,
      onChangeAsync: onChangeFirstName,
    }"
  >
    <template v-slot="{ field, state }">
      <label :htmlFor="field.name">名:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).value)"
        @blur="field.handleBlur"
      />
      <FieldInfo :state="state" />
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

## リアクティビティ

`@tanstack/vue-form` はフォームとフィールドのステート変更を購読するためのさまざまな方法を提供しており、特に `form.useStore` メソッドと `form.Subscribe` コンポーネントが注目されます。これらのメソッドを使用すると、必要な時だけコンポーネントを更新することでフォームのレンダリングパフォーマンスを最適化できます。

例:

```vue
<script setup lang="ts">
// ...
const firstName = form.useStore((state) => state.values.firstName)
</script>

<template>
  <!-- ... -->
  <form.Subscribe>
    <template v-slot="{ canSubmit, isSubmitting }">
      <button type="submit" :disabled="!canSubmit">
        {{ isSubmitting ? '...' : '送信' }}
      </button>
    </template>
  </form.Subscribe>
  <!-- ... -->
</template>
```

## リスナー

`@tanstack/vue-form` では、特定のトリガーに反応して「リスニング」し、副作用をディスパッチすることができます。

例:

```vue
<template>
  <form.Field
    name="country"
    :listeners="{
      onChange: ({ value }) => {
        console.log(`国が ${value} に変更されました、州をリセットします`)
        form.setFieldValue('province', '')
      },
    }"
  >
    <template v-slot="{ field }">
      <input
        :value="field.state.value"
        @input="(e) => field.handleChange(e.target.value)"
      />
    </template>
  </form.Field>
</template>
```

詳細は [リスナー](./listeners.md) を参照してください

注: リアクティビティを実現するための `form.useField` メソッドの使用は推奨されません。このメソッドは `form.Field` コンポーネント内で慎重に使用するように設計されています。代わりに `form.useStore` を使用することを検討してください。

## 配列フィールド

配列フィールドを使用すると、趣味のリストなどのフォーム内の値のリストを管理できます。`mode="array"` プロパティを指定した `form.Field` コンポーネントを使用して配列フィールドを作成できます。

配列フィールドを操作する際には、`pushValue`、`removeValue`、`swapValues`、`moveValue` メソッドを使用して配列内の値を追加、削除、交換できます。

例:

```vue
<template>
  <form @submit.prevent.stop="form.handleSubmit">
    <form.Field name="hobbies" mode="array">
      <template v-slot="{ field: hobbiesField }">
        <div>
          趣味
          <div>
            <div
              v-if="
                Array.isArray(hobbiesField.state.value) &&
                !hobbiesField.state.value.length
              "
            >
              趣味が見つかりません。
            </div>
            <div v-else>
              <div v-for="(_, i) in hobbiesField.state.value" :key="i">
                <form.Field :name="`hobbies[${i}].name`">
                  <template v-slot="{ field }">
                    <div>
                      <label :for="field.name">名前:</label>
                      <input
                        :id="field.name"
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="(e) => field.handleChange(e.target.value)"
                      />
                      <button
                        type="button"
                        @click="hobbiesField.removeValue(i)"
                      >
                        X
                      </button>
                      <FieldInfo :field="field" />
                    </div>
                  </template>
                </form.Field>
                <form.Field :name="`hobbies[${i}].description`">
                  <template v-slot="{ field }">
                    <div>
                      <label :for="field.name">説明:</label>
                      <input
                        :id="field.name"
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="(e) => field.handleChange(e.target.value)"
                      />
                      <FieldInfo :field="field" />
                    </div>
                  </template>
                </form.Field>
              </div>
            </div>
            <button
              type="button"
              @click="
                hobbiesField.pushValue({
                  name: '',
                  description: '',
                  yearsOfExperience: 0,
                })
              "
            >
              趣味を追加
            </button>
          </div>
        </div>
      </template>
    </form.Field>
  </form>
</template>
```

これらは `@tanstack/vue-form` ライブラリで使用される基本概念と用語です。これらの概念を理解することで、ライブラリをより効果的に活用し、複雑なフォームを簡単に作成できるようになります。
