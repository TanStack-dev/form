---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T21:26:51.562Z'
id: form-validation
title: フォームバリデーション
---

TanStack Formの機能の中核にはバリデーションの概念があります。TanStack Formではバリデーションを高度にカスタマイズ可能です:

- バリデーションを実行するタイミングを制御可能（変更時、入力時、フォーカス喪失時、送信時など）
- バリデーションルールはフィールドレベルまたはフォームレベルで定義可能
- 同期バリデーションと非同期バリデーション（API呼び出しの結果など）の両方をサポート

## バリデーションの実行タイミング

実行タイミングは開発者が制御できます！`<Field />`コンポーネントは`onChange`や`onBlur`などのコールバックをpropsとして受け取ります。これらのコールバックにはフィールドの現在値とfieldAPIオブジェクトが渡されるので、バリデーションを実行できます。バリデーションエラーが見つかった場合、単にエラーメッセージを文字列で返すと、`field.state.meta.errors`で利用可能になります。

以下は例です:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

上記の例では、バリデーションは各キーストローク時（`onChange`）に実行されます。代わりにフィールドのフォーカスが外れた時にバリデーションを行いたい場合は、以下のようにコードを変更します:

```tsx
<form.Field
  name="age"
  validators={{
    onBlur: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        // フィールドのonBlurイベントをリッスン
        onBlur={field.handleBlur}
        // TanStack Formが変更を受け取るため、onChangeは常に実装が必要
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

このように、目的のコールバックを実装することでバリデーションのタイミングを制御できます。異なるタイミングで異なるバリデーションを実行することも可能です:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
    onBlur: ({ value }) => (value < 0 ? 'Invalid value' : undefined),
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        // フィールドのonBlurイベントをリッスン
        onBlur={field.handleBlur}
        // TanStack Formが変更を受け取るため、onChangeは常に実装が必要
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

上記の例では、同じフィールドに対して異なるタイミング（各キーストローク時とフィールドのフォーカス喪失時）で異なるバリデーションを行っています。`field.state.meta.errors`は配列なので、特定の時点で関連するすべてのエラーが表示されます。また、`field.state.meta.errorMap`を使用して、バリデーションが行われたタイミング（onChange、onBlurなど）に基づいてエラーを取得することもできます。エラーの表示方法についての詳細は後述します。

## エラーの表示

バリデーションを設定したら、エラーを配列からマッピングしてUIに表示できます:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => {
    return (
      <>
        {/* ... */}
        {!field.state.meta.isValid && (
          <em>{field.state.meta.errors.join(',')}</em>
        )}
      </>
    )
  }}
</form.Field>
```

または、`errorMap`プロパティを使用して特定のエラーにアクセスできます:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => (
    <>
      {/* ... */}
      {field.state.meta.errorMap['onChange'] ? (
        <em>{field.state.meta.errorMap['onChange']}</em>
      ) : null}
    </>
  )}
</form.Field>
```

`errors`配列と`errorMap`はバリデーターが返す型と一致することに注意してください。つまり:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }}
>
  {(field) => (
    <>
      {/* ... */}
      {/* errorMap.onChangeの型は `{isOldEnough: false} | undefined` */}
      {/* meta.errorsの型は `Array<{isOldEnough: false} | undefined>` */}
      {!field.state.meta.errorMap['onChange']?.isOldEnough ? (
        <em>The user is not old enough</em>
      ) : null}
    </>
  )}
</form.Field>
```

## フィールドレベル vs フォームレベルのバリデーション

上記のように、各`<Field>`は`onChange`、`onBlur`などのコールバックを通じて独自のバリデーションルールを受け入れます。また、`useForm()`フックに同様のコールバックを渡すことで、フォームレベル（フィールドごとではなく）でバリデーションルールを定義することも可能です。

例:

```tsx
export default function App() {
  const form = useForm({
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
  })

  // フォームのエラーマップを購読して更新時に再レンダリング
  // または`form.Subscribe`を使用可能
  const formErrorMap = useStore(form.store, (state) => state.errorMap)

  return (
    <div>
      {/* ... */}
      {formErrorMap.onChange ? (
        <div>
          <em>There was an error on the form: {formErrorMap.onChange}</em>
        </div>
      ) : null}
      {/* ... */}
    </div>
  )
}
```

### フォームのバリデーターからフィールドレベルのエラーを設定

フォームのバリデーターからフィールドにエラーを設定できます。これの一般的な使用例は、フォームの`onSubmitAsync`バリデーターで単一のAPIエンドポイントを呼び出してすべてのフィールドをバリデーションすることです。

```tsx
export default function App() {
  const form = useForm({
    defaultValues: {
      age: 0,
      socials: [],
      details: {
        email: '',
      },
    },
    validators: {
      onSubmitAsync: async ({ value }) => {
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

  return (
    <div>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          void form.handleSubmit()
        }}
      >
        <form.Field name="age">
          {(field) => (
            <>
              <label htmlFor={field.name}>Age:</label>
              <input
                id={field.name}
                name={field.name}
                value={field.state.value}
                type="number"
                onChange={(e) => field.handleChange(e.target.valueAsNumber)}
              />
              {!field.state.meta.isValid && (
                <em role="alert">{field.state.meta.errors.join(', ')}</em>
              )}
            </>
          )}
        </form.Field>
        <form.Subscribe
          selector={(state) => [state.errorMap]}
          children={([errorMap]) =>
            errorMap.onSubmit ? (
              <div>
                <em>There was an error on the form: {errorMap.onSubmit}</em>
              </div>
            ) : null
          }
        />
        {/*...*/}
      </form>
    </div>
  )
}
```

> 注意点として、エラーを返すフォームバリデーション関数がある場合、そのエラーはフィールド固有のバリデーションによって上書きされる可能性があります。
>
> つまり:
>
> ```jsx
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
>
> // ...
>
> return (
>   <form.Field
>     name="age"
>     validators={{
>       onChange: ({ value }) => (value % 2 === 0 ? 'Must be odd!' : undefined),
>     }}
>     children={() => <>{/* ... */}</>}
>   />
> )
> ```
>
> この場合、フォームレベルのバリデーションで'Too young!'エラーが返されても、'Must be odd!'のみが表示されます。

## 非同期関数バリデーション

多くのバリデーションは同期処理になると予想されますが、ネットワーク呼び出しやその他の非同期操作に対してバリデーションを行うことが有用な場合も多くあります。

このために、`onChangeAsync`、`onBlurAsync`などの専用メソッドをバリデーションに使用できます:

```tsx
<form.Field
  name="age"
  validators={{
    onChangeAsync: async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value < 13 ? 'You must be 13 to make an account' : undefined
    },
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

同期と非同期のバリデーションは共存可能です。例えば、同じフィールドに`onBlur`と`onBlurAsync`の両方を定義できます:

```tsx
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
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        value={field.state.value}
        type="number"
        onBlur={field.handleBlur}
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

同期バリデーションメソッド（`onBlur`）が最初に実行され、非同期メソッド（`onBlurAsync`）は同期メソッド（`onBlur`）が成功した場合にのみ実行されます。この動作を変更するには、`asyncAlways`オプションを`true`に設定すると、同期メソッドの結果に関係なく非同期メソッドが実行されます。

### 組み込みのデバウンス

データベースに対してバリデーションを行う場合、非同期呼び出しが適切な方法ですが、すべてのキーストロークでネットワークリクエストを実行すると、データベースにDDoS攻撃をかけるようなものです。

代わりに、単一のプロパティを追加することで非同期呼び出しを簡単にデバウンスする方法を提供します:

```tsx
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsync: async ({ value }) => {
      // ...
    },
  }}
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

これにより、すべての非同期呼び出しが500msの遅延でデバウンスされます。このプロパティはバリデーションごとにオーバーライドすることも可能です:

```tsx
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
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

これにより、`onChangeAsync`は1500msごとに実行され、`onBlurAsync`は500msごとに実行されます。

## スキーマライブラリによるバリデーション

関数はバリデーションに対してより柔軟性とカスタマイズ性を提供しますが、少し冗長になる場合があります。これを解決するために、スキーマベースのバリデーションを提供し、簡潔で型安全なバリデーションを大幅に簡単にするライブラリがあります。また、フォーム全体の単一スキーマを定義してフォームレベルに渡すこともでき、エラーは自動的にフィールドに伝播されます。

### 標準スキーマライブラリ

TanStack Formは[Standard Schema仕様](https://github.com/standard-schema/standard-schema)に準拠するすべてのライブラリをネイティブでサポートしており、特に以下が含まれます:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)
- [Effect/Schema](https://effect.website/docs/schema/standard-schema/)

_注意:_ 古いバージョンのスキーマライブラリはStandard Schemaをまだサポートしていない可能性があるため、最新バージョンを使用してください。

> バリデーションでは変換された値は提供されません。詳細は[サブミッション処理](./submission-handling.md)を参照してください。

これらのライブラリのスキーマを使用するには、カスタム関数と同じように`validators` propsに渡します:

```tsx
const userSchema = z.object({
  age: z.number().gte(13, 'You must be 13 to make an account'),
})

function App() {
  const form = useForm({
    defaultValues: {
      age: 0,
    },
    validators: {
      onChange: userSchema,
    },
  })
  return (
    <div>
      <form.Field
        name="age"
        children={(field) => {
          return <>{/* ... */}</>
        }}
      />
    </div>
  )
}
```

フォームレベルとフィールドレベルの非同期バリデーションもサポートされています:

```tsx
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
  children={(field) => {
    return <>{/* ...
```
