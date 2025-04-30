---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T21:27:49.320Z'
id: form-validation
title: フォームバリデーション
---

TanStack Formの機能の中核にはバリデーションの概念があります。TanStack Formではバリデーションを高度にカスタマイズ可能です:

- バリデーションを実行するタイミングを制御可能（変更時、入力時、フォーカス喪失時、送信時など）
- バリデーションルールはフィールドレベルまたはフォームレベルで定義可能
- バリデーションは同期処理または非同期処理（API呼び出しの結果など）に対応

## バリデーションの実行タイミング

実行タイミングは開発者が決定します！ `<Field />` コンポーネントは `onChange` や `onBlur` などのコールバックをpropsとして受け取ります。これらのコールバックには現在のフィールド値と `fieldAPI` オブジェクトが渡されるので、バリデーションを実行できます。バリデーションエラーが見つかった場合、単にエラーメッセージを文字列で返すと、`field.state.meta.errors` で利用可能になります。

以下は例です:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

上記の例では、バリデーションは各キーストローク時（`onChange`）に実行されます。代わりにフィールドのフォーカスが外れた時にバリデーションを行いたい場合は、以下のようにコードを変更します:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onBlur: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <!-- TanStack Formが変更を受け取るため、常にonChangeを実装する必要があります -->
      <!-- フィールドのonBlurイベントをリッスン -->
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

このように、必要なコールバックを実装することでバリデーションの実行タイミングを制御できます。異なるタイミングで異なるバリデーションを実行することも可能です:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
      onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <!-- TanStack Formが変更を受け取るため、常にonChangeを実装する必要があります -->
      <!-- フィールドのonBlurイベントをリッスン -->
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="(e) => field.handleChange((e.target as HTMLInputElement).valueAsNumber)
                "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

上記の例では、同じフィールドに対して異なるタイミング（各キーストローク時とフィールドのフォーカス喪失時）で異なるバリデーションを行っています。`field.state.meta.errors` は配列なので、特定の時点で関連するすべてのエラーが表示されます。また、`field.state.meta.errorMap` を使用して、バリデーションが行われたタイミング（onChange、onBlurなど）に基づいてエラーを取得することもできます。エラー表示に関する詳細は後述します。

## エラーの表示

バリデーションを設定したら、エラーを配列からマップしてUIに表示できます:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

または、`errorMap` プロパティを使用して特定のエラーにアクセスすることもできます:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: ({ value }) =>
        value < 13 ? 'You must be 13 to make an account' : undefined,
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
      <em role="alert" v-if="field.state.meta.errorMap['onChange']">{{
        field.state.meta.errorMap['onChange']
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

`errors` 配列と `errorMap` はバリデータによって返される型と一致することに注意してください。つまり:

```vue
<form.Field
  name="age"
  :validators="{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }"
>
  <template v-slot="{ field }">
      <!-- ... -->
      <!-- errorMap.onChangeの型は `{isOldEnough: false} | undefined` -->
	  <!-- meta.errorsの型は `Array<{isOldEnough: false} | undefined>` -->
      <em v-if="!field.state.meta.errorMap['onChange']?.isOldEnough">The user is not old enough</em>
  </template>
</form.Field>
```

## フィールドレベル vs フォームレベルのバリデーション

上記のように、各 `<Field>` は `onChange`、`onBlur` などのコールバックを通じて独自のバリデーションルールを受け入れます。また、`useForm()` 関数に同様のコールバックを渡すことで、フォームレベル（フィールドごとではなく）でバリデーションルールを定義することも可能です。

例:

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    age: 0,
  },
  onSubmit: async ({ value }) => {
    console.log(value)
  },
  validators: {
    // フィールドに追加するのと同じ方法でフォームにバリデータを追加
    onChange({ value }) {
      if (value.age < 13) {
        return 'Must be 13 or older to sign'
      }
      return undefined
    },
  },
})

// フォームのエラーマップを購読して更新時にレンダリング
// または、`form.Subscribe`を使用可能
const formErrorMap = form.useStore((state) => state.errorMap)
</script>

<template>
  <!-- ... -->
  <div v-if="formErrorMap.onChange">
    <em role="alert">
      There was an error on the form: {{ formErrorMap.onChange }}
    </em>
  </div>
  <!-- ... -->
</template>
```

### フォームのバリデータからフィールドレベルのエラーを設定

フォームのバリデータからフィールドにエラーを設定できます。これの一般的な使用例は、フォームの `onSubmitAsync` バリデータで単一のAPIエンドポイントを呼び出してすべてのフィールドをバリデーションすることです。

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    age: 0,
    socials: [],
    details: {
      email: '',
    },
  },
  validators: {
    // フィールドに追加するのと同じ方法でフォームにバリデータを追加
    onSubmitAsync({ value }) {
      // サーバーで値をバリデーション
      const hasErrors = await verifyDataOnServer(value)
      if (hasErrors) {
        return {
          form: 'Invalid data', // `form`キーはオプション
          fields: {
            age: 'Must be 13 or older to sign',
            // フィールド名でネストしたフィールドにエラーを設定
            'socials[0].url': 'The provided URL does not exist',
            'details.email': 'An email is required',
          },
        }
      }

      return null
    },
  },
})
</script>

<template>
  <!-- ... -->
</template>
```

> 注意点として、フォームバリデーション関数がエラーを返した場合、そのエラーはフィールド固有のバリデーションによって上書きされる可能性があります。
>
> つまり:
>
> ```vue
> <script setup lang="ts">
> import { useForm } from '@tanstack/vue-form'
>
> const form = useForm({
>   defaultValues: {
>     age: 0,
>   },
>   validators: {
>     onChange: ({ value }) => {
>       return {
>         fields: {
>           age: value.age < 12 ? 'Too young!' : undefined,
>         },
>       }
>     },
>   },
> })
> </script>
>
> <template>
>   <!-- ... -->
>   <form.Field
>     name="age"
>     :validators="{
>       onChange: ({ value }) => (value % 2 === 0 ? 'Must be odd!' : undefined),
>     }"
>   >
>     <template v-slot="{ field }">
>       <!-- ... -->
>     </template>
>   </form.Field>
> </template>
> ```
>
> フォームレベルのバリデーションで 'Too young!' エラーが返されても、'Must be odd!' のみが表示されます。

## 非同期関数バリデーション

ほとんどのバリデーションは同期処理になると予想されますが、ネットワーク呼び出しやその他の非同期操作に対してバリデーションを行うことが有用な場合も多くあります。

このために、`onChangeAsync`、`onBlurAsync` などの専用メソッドを使用できます:

```vue
<script setup lang="ts">
// ...

const onChangeAge = async ({ value }) => {
  await new Promise((resolve) => setTimeout(resolve, 1000))
  return value < 13 ? 'You must be 13 to make an account' : undefined
}
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChangeAsync: onChangeAge,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @input="
          (e) =>
            field.handleChange((e.target as HTMLInputElement).valueAsNumber)
        "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

同期と非同期のバリデーションは共存可能です。例えば、同じフィールドに `onBlur` と `onBlurAsync` の両方を定義できます:

```vue
<script setup lang="ts">
// ...

const onBlurAge = ({ value }) => (value < 0 ? 'Invalid value' : undefined)

const onBlurAgeAsync = async ({ value }) => {
  const currentAge = await fetchCurrentAgeOnProfile()
  return value < currentAge ? 'You can only increase the age' : undefined
}
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onBlur: onBlurAge,
      onBlurAsync: onBlurAgeAsync,
    }"
  >
    <template v-slot="{ field }">
      <label :for="field.name">Age:</label>
      <input
        :id="field.name"
        :name="field.name"
        :value="field.state.value"
        type="number"
        @blur="field.handleBlur"
        @input="
          (e) =>
            field.handleChange((e.target as HTMLInputElement).valueAsNumber)
        "
      />
      <em role="alert" v-if="!field.state.meta.isValid">{{
        field.state.meta.errors.join(', ')
      }}</em>
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

同期バリデーションメソッド（`onBlur`）が最初に実行され、非同期メソッド（`onBlurAsync`）は同期メソッド（`onBlur`）が成功した場合にのみ実行されます。この動作を変更するには、`asyncAlways` オプションを `true` に設定すると、同期メソッドの結果に関係なく非同期メソッドが実行されます。

### 組み込みのデバウンス

非同期呼び出しはデータベースに対してバリデーションを行う際に適していますが、キーストロークごとにネットワークリクエストを実行すると、データベースをDDoS攻撃する良い方法になってしまいます。

代わりに、単一のプロパティを追加するだけで非同期呼び出しをデバウンスする簡単な方法を提供します:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :async-debounce-ms="500"
    :validators="{
      onChangeAsync: async ({ value }) => {
        // ...
      },
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

これにより、すべての非同期呼び出しが500msの遅延でデバウンスされます。このプロパティはバリデーションごとにオーバーライドすることも可能です:

```vue
<template>
  <!-- ... -->
  <form.Field
    name="age"
    :async-debounce-ms="500"
    :validators="{
      onChangeAsyncDebounceMs: 1500,
      onChangeAsync: async ({ value }) => {
        // ...
      },
      onBlurAsync: async ({ value }) => {
        // ...
      },
    }"
  >
    <template v-slot="{ field }">
      <!-- ... -->
    </template>
  </form.Field>
  <!-- ... -->
</template>
```

これにより、`onChangeAsync` は1500msごとに、`onBlurAsync` は500msごとに実行されます。

## スキーマライブラリによるバリデーション

関数はバリデーションに対してより柔軟性とカスタマイズ性を提供しますが、少し冗長になる可能性があります。これを解決するために、簡潔で型安全なバリデーションを大幅に簡単にするスキーマベースのバリデーションを提供するライブラリがあります。また、フォーム全体に対して単一のスキーマを定義し、フォームレベルに渡すことも可能で、エラーは自動的にフィールドに伝搬されます。

### 標準スキーマライブラリ

TanStack Formは [Standard Schema仕様](https://github.com/standard-schema/standard-schema) に準拠するすべてのライブラリをネイティブでサポートしており、特に以下が含まれます:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_注:_ 古いバージョンはStandard Schemaをまだサポートしていない可能性があるため、スキーマライブラリの最新バージョンを使用してください。

> バリデーションは変換された値を提供しません。詳細は [送信処理](./submission-handling.md) を参照してください。

これらのライブラリからのスキーマを使用するには、カスタム関数と同じように `validators` props に渡します:

```vue
<script setup lang="ts">
import { z } from 'zod'
// ...

const form = useForm({
  // ...
})
</script>

<template>
  <!-- ... -->
  <form.Field
    name="age"
    :validators="{
      onChange: z.number().gte(13, 'You must be 13 to make an account'),
    }"
  >
    <template v-slot="{ field }">
      <!-- ...
```
