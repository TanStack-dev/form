---
source-updated-at: '2025-03-04T14:23:57.000Z'
translation-updated-at: '2025-04-30T21:26:01.377Z'
id: custom-errors
title: カスタムエラー
---

# カスタムエラー

TanStack Formでは、バリデーターから返すエラー値のタイプに完全な柔軟性を提供します。文字列のエラーが最も一般的で扱いやすいですが、ライブラリではバリデーターからあらゆるタイプの値を返すことが可能です。

一般的なルールとして、truthyな値はエラーと見なされ、フォームまたはフィールドを無効とマークします。一方、falsyな値（`false`、`undefined`、`null`など）はエラーが存在せず、フォームまたはフィールドが有効であることを意味します。

## フォームから文字列値を返す

```tsx
<form.Field
  name="username"
  validators={{
    onChange: ({ value }) =>
      value.length < 3 ? 'Username must be at least 3 characters' : undefined,
  }}
/>
```

複数のフィールドに影響するフォームレベルのバリデーション:

```tsx
const form = useForm({
  defaultValues: {
    username: '',
    email: '',
  },
  validators: {
    onChange: ({ value }) => {
      return {
        fields: {
          username:
            value.username.length < 3 ? 'Username too short' : undefined,
          email: !value.email.includes('@') ? 'Invalid email' : undefined,
        },
      }
    },
  },
})
```

文字列エラーは最も一般的なタイプで、UIに簡単に表示できます:

```tsx
{
  field.state.meta.errors.map((error, i) => (
    <div key={i} className="error">
      {error}
    </div>
  ))
}
```

### 数値

数量、閾値、大きさを表すのに有用:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 18 ? 18 - value : undefined),
  }}
/>
```

UIでの表示:

```tsx
{
  /* TypeScriptはバリデーターに基づいてエラーが数値であることを認識 */
}
;<div className="error">
  資格を得るにはあと{field.state.meta.errors[0]}年必要です
</div>
```

### ブール値

エラー状態を示す単純なフラグ:

```tsx
<form.Field
  name="accepted"
  validators={{
    onChange: ({ value }) => (!value ? true : undefined),
  }}
/>
```

UIでの表示:

```tsx
{
  field.state.meta.errors[0] === true && (
    <div className="error">利用規約に同意する必要があります</div>
  )
}
```

### オブジェクト

複数のプロパティを持つリッチなエラーオブジェクト:

```tsx
<form.Field
  name="email"
  validators={{
    onChange: ({ value }) => {
      if (!value.includes('@')) {
        return {
          message: '無効なメール形式',
          severity: 'error',
          code: 1001,
        }
      }
      return undefined
    },
  }}
/>
```

UIでの表示:

```tsx
{
  typeof field.state.meta.errors[0] === 'object' && (
    <div className={`error ${field.state.meta.errors[0].severity}`}>
      {field.state.meta.errors[0].message}
      <small> (コード: {field.state.meta.errors[0].code})</small>
    </div>
  )
}
```

上の例では、表示したいイベントエラーに依存します。

### 配列

単一フィールドに対する複数のエラーメッセージ:

```tsx
<form.Field
  name="password"
  validators={{
    onChange: ({ value }) => {
      const errors = []
      if (value.length < 8) errors.push('パスワードが短すぎます')
      if (!/[A-Z]/.test(value)) errors.push('大文字が含まれていません')
      if (!/[0-9]/.test(value)) errors.push('数字が含まれていません')

      return errors.length ? errors : undefined
    },
  }}
/>
```

UIでの表示:

```tsx
{
  Array.isArray(field.state.meta.errors) && (
    <ul className="error-list">
      {field.state.meta.errors.map((err, i) => (
        <li key={i}>{err}</li>
      ))}
    </ul>
  )
}
```

## フィールドの`disableErrorFlat`プロパティ

デフォルトでは、TanStack Formはすべてのバリデーションソース（onChange、onBlur、onSubmit）からのエラーを単一の`errors`配列に平坦化します。`disableErrorFlat`プロパティを使用すると、エラーソースを保持できます:

```tsx
<form.Field
  name="email"
  disableErrorFlat
  validators={{
    onChange: ({ value }) =>
      !value.includes('@') ? '無効なメール形式' : undefined,
    onBlur: ({ value }) =>
      !value.endsWith('.com') ? '.comドメインのみ許可されています' : undefined,
    onSubmit: ({ value }) =>
      value.length < 5 ? 'メールが短すぎます' : undefined,
  }}
/>
```

`disableErrorFlat`なしでは、すべてのエラーは`field.state.meta.errors`に結合されます。これを使用すると、ソースごとにエラーにアクセスできます:

```tsx
{
  field.state.meta.errorMap.onChange && (
    <div className="real-time-error">{field.state.meta.errorMap.onChange}</div>
  )
}

{
  field.state.meta.errorMap.onBlur && (
    <div className="blur-feedback">{field.state.meta.errorMap.onBlur}</div>
  )
}

{
  field.state.meta.errorMap.onSubmit && (
    <div className="submit-error">{field.state.meta.errorMap.onSubmit}</div>
  )
}
```

これは以下に有用です:

- 異なるUI処理で異なるタイプのエラーを表示
- エラーの優先順位付け（例: 送信エラーをより目立たせる）
- エラーの段階的開示の実装

## `errors`と`errorMap`の型安全性

TanStack Formはエラーハンドリングに強力な型安全性を提供します。`errorMap`の各キーは対応するバリデーターから返される型と正確に一致し、`errors`配列にはすべてのバリデーターからの可能なエラー値のユニオン型が含まれます:

```tsx
<form.Field
  name="password"
  validators={{
    onChange: ({ value }) => {
      // これは文字列またはundefinedを返す
      return value.length < 8 ? '短すぎます' : undefined
    },
    onBlur: ({ value }) => {
      // これはオブジェクトまたはundefinedを返す
      if (!/[A-Z]/.test(value)) {
        return { message: '大文字がありません', level: 'warning' }
      }
      return undefined
    },
  }}
  children={(field) => {
    // TypeScriptはerrors[0]が string | { message: string, level: string } | undefined になり得ることを認識
    const error = field.state.meta.errors[0]

    // 型安全なエラーハンドリング
    if (typeof error === 'string') {
      return <div className="string-error">{error}</div>
    } else if (error && typeof error === 'object') {
      return <div className={error.level}>{error.message}</div>
    }

    return null
  }}
/>
```

`errorMap`プロパティも完全に型付けされ、バリデーション関数の戻り値の型と一致します:

```tsx
// disableErrorFlat使用時
<form.Field
  name="email"
  disableErrorFlat
  validators={{
    onChange: ({ value }): string | undefined =>
      !value.includes("@") ? "無効なメール" : undefined,
    onBlur: ({ value }): { code: number, message: string } | undefined =>
      !value.endsWith(".com") ? { code: 100, message: "不正なドメイン" } : undefined
  }}
  children={(field) => {
    // TypeScriptは各エラーソースの正確な型を知っている
    const onChangeError: string | undefined = field.state.meta.errorMap.onChange;
    const onBlurError: { code: number, message: string } | undefined = field.state.meta.errorMap.onBlur;

    return (/* ... */);
  }}
/>
```

この型安全性により、ランタイムではなくコンパイル時にエラーを検出できるため、コードの信頼性と保守性が向上します。
