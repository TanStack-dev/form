---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:29:34.245Z'
id: form-validation
title: フォームバリデーション
---

TanStack Formの機能の中核となるのがバリデーションの概念です。TanStack Formではバリデーションを高度にカスタマイズ可能です:

- バリデーションを実行するタイミングを制御可能（変更時、入力時、フォーカス喪失時、送信時など）
- バリデーションルールはフィールドレベルまたはフォームレベルで定義可能
- 同期バリデーションと非同期バリデーション（API呼び出しの結果など）の両方をサポート

## バリデーションの実行タイミング

実行タイミングは開発者が制御できます！`<form.Field />`コンポーネントは`onChange`や`onBlur`などのコールバックをpropsとして受け取ります。これらのコールバックにはフィールドの現在値とfieldAPIオブジェクトが渡されるので、バリデーションを実行できます。バリデーションエラーが見つかった場合、単にエラーメッセージを文字列で返すと、`field.state.meta.errors`で利用可能になります。

以下は例です:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

上記の例では、バリデーションは各キーストローク時（`onchange`）に実行されます。代わりにフィールドのフォーカスが外れた時にバリデーションを行いたい場合は、以下のようにコードを変更します:

```svelte
<form.Field
  name="age"
  validators={{
    onBlur: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

このように、目的のコールバックを実装することでバリデーションのタイミングを制御できます。異なるタイミングで異なるバリデーションを実行することも可能です:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
    onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

上記の例では、同じフィールドに対して異なるタイミング（各キーストローク時とフィールドのフォーカス喪失時）で異なるバリデーションを行っています。`field.state.meta.errors`は配列なので、特定の時点で関連するすべてのエラーが表示されます。また、`field.state.meta.errorMap`を使用して、バリデーションが行われたタイミング（onChange、onBlurなど）に基づいてエラーを取得することもできます。エラーの表示方法についての詳細は後述します。

## エラーの表示

バリデーションを設定したら、エラーを配列からマッピングしてUIに表示できます:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

または、`errorMap`プロパティを使用して特定のエラーにアクセスすることもできます:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    {#if field.state.meta.errorMap['onChange']}
      <em role="alert">{field.state.meta.errorMap['onChange']}</em>
    {/if}
  {/snippet}
</form.Field>
```

`errors`配列と`errorMap`はバリデーターが返す型と一致することに注意してください。つまり:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    <!-- errorMap.onChangeは型 `{isOldEnough: false} | undefined` -->
    <!-- meta.errorsは型 `Array<{isOldEnough: false} | undefined>` -->
    {#if field.state.meta.errorMap['onChange']?.isOldEnough}
        <em>The user is not old enough</em>
    {/if}
  {/snippet}
</form.Field>
```

## フィールドレベル vs フォームレベルのバリデーション

上記のように、各`<form.Field>`は`onChange`、`onBlur`などのコールバックを通じて独自のバリデーションルールを受け取ります。また、`createForm()`フックに同様のコールバックを渡すことで、フォームレベル（フィールドごとではなく）でバリデーションルールを定義することも可能です。

例:

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      age: 0,
    },
    onSubmit: async ({ value }) => {
      console.log(value)
    },
    validators: {
      // フィールドに追加するのと同じ方法でフォームにバリデーターを追加
      onChange({ value }) {
        if (value.age < 13) {
          return 'Must be 13 or older to sign'
        }
        return undefined
      },
    },
  }))

  // フォームのエラーマップを購読して更新時に再レンダリング
  // または、`form.Subscribe`を使用可能
  const formErrorMap = form.useStore((state) => state.errorMap)
</script>

<div>
  <!-- ... -->
  {#if formErrorMap.current.onChange}
    <div>
      <em>There was an error on the form: {formErrorMap.current.onChange}</em>
    </div>
  {/if}
  <!-- ... -->
</div>
```

## 非同期関数型バリデーション

多くのバリデーションは同期処理になると予想されますが、ネットワーク呼び出しやその他の非同期操作に対してバリデーションを行うことが有用な場合も多くあります。

このために、`onChangeAsync`、`onBlurAsync`などの専用メソッドをバリデーションに使用できます:

```svelte
<form.Field
  name="age"
  validators={{
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value < 13 ? 'You must be 13 to make an account' : undefined
    },
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

同期バリデーションと非同期バリデーションは共存可能です。例えば、同じフィールドに`onBlur`と`onBlurAsync`の両方を定義できます:

```svelte
<form.Field
  name="age"
  validators={{
    onBlur: ({ value }) => (value < 13 ? 'You must be at least 13' : undefined),
    onBlurAsync: async ({ value }) => {
      const currentAge = await fetchCurrentAgeOnProfile()
      return value < currentAge ? 'You can only increase the age' : undefined
    },
  }}
>
  {#snippet children(field)}
    <label for={field.name}>Age:</label>
    <input
      id={field.name}
      name={field.name}
      value={field.state.value}
      type="number"
      onblur={field.handleBlur}
      onchange={(e) => field.handleChange(e.target.valueAsNumber)}
    />
    {#if field.state.meta.errors}
      <em role="alert">{field.state.meta.errors.join(', ')}</em>
    {/if}
  {/snippet}
</form.Field>
```

同期バリデーションメソッド（`onBlur`）が最初に実行され、非同期メソッド（`onBlurAsync`）は同期メソッド（`onBlur`）が成功した場合にのみ実行されます。この動作を変更するには、`asyncAlways`オプションを`true`に設定すると、同期メソッドの結果に関係なく非同期メソッドが実行されます。

### 組み込みのデバウンス

非同期呼び出しはデータベースに対するバリデーションを行う際に有効ですが、すべてのキーストロークでネットワークリクエストを実行すると、データベースにDDoS攻撃を仕掛けるようなものです。

代わりに、単一のプロパティを追加するだけで非同期呼び出しをデバウンスする簡単な方法を提供しています:

```svelte
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsync: async ({ value }) => {
      // ...
    },
  }}
>
  <!-- ... -->
</form.Field>
```

これにより、すべての非同期呼び出しが500msの遅延でデバウンスされます。バリデーションプロパティごとにこのプロパティをオーバーライドすることも可能です:

```svelte
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsyncDebounceMs: 1500,
    onChangeAsync: async ({ value }) => {
      // ...
    },
    onBlurAsync: async ({ value }) => {
      // ...
    },
  }}
>
  <!-- ... -->
</form.Field>
```

> これにより、`onChangeAsync`は1500msごとに実行され、`onBlurAsync`は500msごとに実行されます。

## スキーマライブラリによるバリデーション

関数はバリデーションに対してより柔軟性とカスタマイズ性を提供しますが、少し冗長になる可能性があります。これを解決するために、スキーマベースのバリデーションを提供し、簡潔で型に厳密なバリデーションを大幅に容易にするライブラリがあります。また、フォーム全体に対して単一のスキーマを定義し、フォームレベルに渡すことも可能で、エラーは自動的にフィールドに伝播されます。

### 標準スキーマライブラリ

TanStack Formは[Standard Schema仕様](https://github.com/standard-schema/standard-schema)に準拠するすべてのライブラリをネイティブでサポートしており、特に以下のライブラリが有名です:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_注意:_ 古いバージョンはStandard Schemaをまだサポートしていない可能性があるため、スキーマライブラリの最新バージョンを使用してください。

これらのライブラリからのスキーマを使用するには、カスタム関数と同じように`validators` propsに渡します:

```svelte
<script>
  import { z } from 'zod'

  // ...

  const form = createForm(() => ({
    // ...
  }))
</script>

<form.Field
  name="age"
  validators={{
    onChange: z.number().gte(13, 'You must be 13 to make an account'),
  }}
>
  <!-- ... -->
</form.Field>
```

フォームレベルとフィールドレベルの非同期バリデーションもサポートされています:

```svelte
<form.Field
  name="age"
  validators={{
    onChange: z.number().gte(13, 'You must be 13 to make an account'),
    onChangeAsyncDebounceMs: 500,
    onChangeAsync: z.number().refine(
      async (value) => {
        const currentAge = await fetchCurrentAgeOnProfile()
        return value >= currentAge
      },
      {
        message: 'You can only increase the age',
      },
    ),
  }}
>
  <!-- ... -->
</form.Field>
```

## 無効なフォームの送信を防止

`onChange`、`onBlur`などのコールバックはフォームが送信される時にも実行され、フォームが無効な場合、送信はブロックされます。

フォーム状態オブジェクトには`canSubmit`フラグがあり、いずれかのフィールドが無効でフォームが操作された場合（`canSubmit`はフォームが操作されるまではtrueのままです。`onChange`/`onBlur` propsに基づいていくつかのフィールドが「技術的に」無効であっても）falseになります。

`form.Subscribe`を介してこれを購読し、例えばフォームが無効な場合に送信ボタンを無効化する（実際には、無効化されたボタンはアクセシブルではないため、代わりに`aria-disabled`を使用する）ために値を使用できます。

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    /* ... */
  }))
</script>

<!-- ... -->

<!-- 動的な送信ボタン -->
<form.Subscribe
  selector={(state) => ({
    canSubmit: state.canSubmit,
    isSubmitting: state.isSubmitting,
  })}
  children={(state) => (
    <button type="submit" disabled={!state().canSubmit}>
      {state().isSubmitting ? '...' : 'Submit'}
    </button>
  )}
>
  {#snippet children(state)}
    <button type="submit" disabled={!state.canSubmit}>
      {state.isSubmitting ? '...' : 'Submit'}
    </button>
  {/snippet}
</form.Subscribe>
```
