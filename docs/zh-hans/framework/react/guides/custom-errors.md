---
source-updated-at: '2025-03-04T14:23:57.000Z'
translation-updated-at: '2025-04-12T04:09:56.557Z'
id: custom-errors
title: 自定义错误
---

TanStack Form 为验证器返回的错误值类型提供了完全的灵活性。字符串错误是最常见且易于处理的类型，但该库允许您从验证器中返回任何类型的值。

作为通用规则，任何真值（truthy value）都会被视作错误，并将表单或字段标记为无效；而假值（falsy values，如 `false`、`undefined`、`null` 等）则表示没有错误，表单或字段有效。

## 从表单返回字符串值

```tsx
<form.Field
  name="username"
  validators={{
    onChange: ({ value }) =>
      value.length < 3 ? '用户名至少需要 3 个字符' : undefined,
  }}
/>
```

对于影响多个字段的表单级验证：

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
          username: value.username.length < 3 ? '用户名过短' : undefined,
          email: !value.email.includes('@') ? '无效的邮箱地址' : undefined,
        },
      }
    },
  },
})
```

字符串错误是最常见的类型，可以轻松地在 UI 中显示：

```tsx
{
  field.state.meta.errors.map((error, i) => (
    <div key={i} className="error">
      {error}
    </div>
  ))
}
```

### 数字类型

适用于表示数量、阈值或量级：

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 18 ? 18 - value : undefined),
  }}
/>
```

在 UI 中显示：

```tsx
{
  /* TypeScript 会根据验证器推断错误类型为数字 */
}
;<div className="error">您还需要 {field.state.meta.errors[0]} 年才符合资格</div>
```

### 布尔类型

简单的标志位表示错误状态：

```tsx
<form.Field
  name="accepted"
  validators={{
    onChange: ({ value }) => (!value ? true : undefined),
  }}
/>
```

在 UI 中显示：

```tsx
{
  field.state.meta.errors[0] === true && (
    <div className="error">必须接受条款</div>
  )
}
```

### 对象类型

包含多个属性的丰富错误对象：

```tsx
<form.Field
  name="email"
  validators={{
    onChange: ({ value }) => {
      if (!value.includes('@')) {
        return {
          message: '无效的邮箱格式',
          severity: 'error',
          code: 1001,
        }
      }
      return undefined
    },
  }}
/>
```

在 UI 中显示：

```tsx
{
  typeof field.state.meta.errors[0] === 'object' && (
    <div className={`error ${field.state.meta.errors[0].severity}`}>
      {field.state.meta.errors[0].message}
      <small> (错误码: {field.state.meta.errors[0].code})</small>
    </div>
  )
}
```

上例中具体显示取决于您想要展示的事件错误类型。

### 数组类型

单个字段的多条错误信息：

```tsx
<form.Field
  name="password"
  validators={{
    onChange: ({ value }) => {
      const errors = []
      if (value.length < 8) errors.push('密码过短')
      if (!/[A-Z]/.test(value)) errors.push('缺少大写字母')
      if (!/[0-9]/.test(value)) errors.push('缺少数字')

      return errors.length ? errors : undefined
    },
  }}
/>
```

在 UI 中显示：

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

## 字段的 `disableErrorFlat` 属性

默认情况下，TanStack Form 会将所有验证来源（onChange、onBlur、onSubmit）的错误合并到单个 `errors` 数组中。`disableErrorFlat` 属性可以保留错误来源：

```tsx
<form.Field
  name="email"
  disableErrorFlat
  validators={{
    onChange: ({ value }) =>
      !value.includes('@') ? '无效的邮箱格式' : undefined,
    onBlur: ({ value }) =>
      !value.endsWith('.com') ? '仅允许 .com 域名' : undefined,
    onSubmit: ({ value }) => (value.length < 5 ? '邮箱地址过短' : undefined),
  }}
/>
```

不使用 `disableErrorFlat` 时，所有错误会被合并到 `field.state.meta.errors`。启用后，可以通过来源访问错误：

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

这个特性适用于：

- 对不同类型错误展示不同的 UI 处理方式
- 错误优先级排序（如更突出显示提交错误）
- 实现错误的渐进式披露

## `errors` 和 `errorMap` 的类型安全

TanStack Form 为错误处理提供了强类型支持。`errorMap` 中的每个键都严格对应其验证器的返回类型，而 `errors` 数组则包含所有验证器可能返回的错误值的联合类型：

```tsx
<form.Field
  name="password"
  validators={{
    onChange: ({ value }) => {
      // 返回字符串或 undefined
      return value.length < 8 ? '过短' : undefined
    },
    onBlur: ({ value }) => {
      // 返回对象或 undefined
      if (!/[A-Z]/.test(value)) {
        return { message: '缺少大写字母', level: 'warning' }
      }
      return undefined
    },
  }}
  children={(field) => {
    // TypeScript 知道 errors[0] 可能是 string | { message: string, level: string } | undefined
    const error = field.state.meta.errors[0]

    // 类型安全的错误处理
    if (typeof error === 'string') {
      return <div className="string-error">{error}</div>
    } else if (error && typeof error === 'object') {
      return <div className={error.level}>{error.message}</div>
    }

    return null
  }}
/>
```

`errorMap` 属性也完全类型化，与验证函数的返回类型匹配：

```tsx
// 使用 disableErrorFlat
<form.Field
  name="email"
  disableErrorFlat
  validators={{
    onChange: ({ value }): string | undefined =>
      !value.includes("@") ? "无效的邮箱" : undefined,
    onBlur: ({ value }): { code: number, message: string } | undefined =>
      !value.endsWith(".com") ? { code: 100, message: "错误的域名" } : undefined
  }}
  children={(field) => {
    // TypeScript 知道每个错误来源的确切类型
    const onChangeError: string | undefined = field.state.meta.errorMap.onChange;
    const onBlurError: { code: number, message: string } | undefined = field.state.meta.errorMap.onBlur;

    return (/* ... */);
  }}
/>
```

这种类型安全机制帮助您在编译时而非运行时捕获错误，使代码更加可靠和可维护。
