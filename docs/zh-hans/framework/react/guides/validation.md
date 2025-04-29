---
source-updated-at: '2025-04-16T14:15:06.000Z'
translation-updated-at: '2025-04-29T23:16:17.934Z'
id: form-validation
title: 表单验证
---

表单与字段验证 (Form and Field Validation)

TanStack Form 的核心功能之一是验证机制。它提供了高度可定制的验证方式：

- 可自由控制验证触发时机（值变更时、输入时、失焦时、提交时等）
- 验证规则可在字段级别或表单级别定义
- 支持同步验证和异步验证（例如通过 API 调用返回结果）

## 验证触发时机

完全由您决定！`<Field />` 组件接受如 `onChange` 或 `onBlur` 等回调函数作为属性。这些回调会接收字段当前值和 fieldAPI 对象，便于执行验证。若发现验证错误，只需返回字符串类型的错误信息，该信息会存储在 `field.state.meta.errors` 中。

示例：

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
      {field.state.meta.errors ? (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

上例在每次按键时触发验证（`onChange`）。若改为失焦时验证，代码调整如下：

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
        // 监听字段的失焦事件
        onBlur={field.handleBlur}
        // 仍需实现 onChange 以保证 TanStack Form 能接收变更
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {field.state.meta.errors ? (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

通过实现不同的回调函数，您可以灵活控制验证时机。甚至可以在不同时机执行不同验证：

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
        // 监听字段的失焦事件
        onBlur={field.handleBlur}
        // 仍需实现 onChange 以保证 TanStack Form 能接收变更
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
      />
      {field.state.meta.errors ? (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

上例在同一字段的不同时机（按键时和失焦时）执行不同验证。由于 `field.state.meta.errors` 是数组，会显示当前所有相关错误。也可使用 `field.state.meta.errorMap` 根据验证时机（onChange、onBlur 等）获取错误。下文将详细介绍错误展示。

## 错误展示

配置验证后，可将错误数组映射到界面展示：

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
        {field.state.meta.errors.length ? (
          <em>{field.state.meta.errors.join(',')}</em>
        ) : null}
      </>
    )
  }}
</form.Field>
```

或使用 `errorMap` 属性获取特定错误：

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

需注意，`errors` 数组和 `errorMap` 的类型与验证器返回类型匹配。例如：

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
      {/* errorMap.onChange 类型为 `{isOldEnough: false} | undefined` */}
      {/* meta.errors 类型为 `Array<{isOldEnough: false} | undefined>` */}
      {!field.state.meta.errorMap['onChange']?.isOldEnough ? (
        <em>The user is not old enough</em>
      ) : null}
    </>
  )}
</form.Field>
```

## 字段级验证 vs 表单级验证

如上所示，每个 `<Field>` 通过 `onChange`、`onBlur` 等回调接受自己的验证规则。也可以通过向 `useForm()` 钩子传递类似回调来定义表单级验证规则（而非逐个字段定义）。

示例：

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
      // 以与字段相同的方式向表单添加验证器
      onChange({ value }) {
        if (value.age < 13) {
          return 'Must be 13 or older to sign'
        }
        return undefined
      },
    },
  })

  // 订阅表单的错误映射以触发更新
  // 也可使用 `form.Subscribe`
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

### 通过表单验证器设置字段级错误

可通过表单验证器设置字段错误。常见用例是在表单的 `onSubmitAsync` 验证器中调用单个 API 端点来验证所有字段。

```tsx
export default function App() {
  const form = useForm({
    defaultValues: {
      age: 0,
    },
    validators: {
      onSubmitAsync: async ({ value }) => {
        // 在服务端验证年龄
        const isOlderThan13 = await verifyAgeOnServer(value.age)
        if (!isOlderThan13) {
          return {
            form: 'Invalid data', // `form` 键是可选的
            fields: {
              age: 'Must be 13 or older to sign',
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
              {field.state.meta.errors ? (
                <em role="alert">{field.state.meta.errors.join(', ')}</em>
              ) : null}
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

> 需注意：如果表单验证函数返回错误，该错误可能会被字段特定验证覆盖。
>
> 这意味着：
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
>   />
> )
> ```
>
> 即使表单级验证返回 'Too young!' 错误，也只会显示 `'Must be odd!'`。

## 异步函数验证

虽然大多数验证可能是同步的，但许多场景下需要通过网络调用或其他异步操作进行验证。

为此，我们提供了专用的 `onChangeAsync`、`onBlurAsync` 等方法：

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
      {field.state.meta.errors ? (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

同步和异步验证可以共存。例如，同一字段可同时定义 `onBlur` 和 `onBlurAsync`：

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
      {field.state.meta.errors ? (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

同步验证方法（`onBlur`）会先执行，只有同步验证成功时才会执行异步方法（`onBlurAsync`）。要改变此行为，可将 `asyncAlways` 选项设为 `true`，这样无论同步方法结果如何都会执行异步方法。

### 内置防抖

虽然异步验证是与数据库交互的正确方式，但每次按键都发起网络请求可能导致数据库遭受 DDoS 攻击。

我们提供了简单的方法来防抖异步调用，只需添加一个属性：

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

这将为所有异步调用添加 500 毫秒防抖延迟。甚至可以针对特定验证属性单独覆盖：

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

> 这将使 `onChangeAsync` 每 1500 毫秒执行一次，而 `onBlurAsync` 仍保持 500 毫秒间隔。

## 通过模式库验证

虽然函数提供了更灵活的验证方式，但可能显得冗长。为解决此问题，有些库提供基于模式的验证，能大幅简化类型严格的验证。您还可以为整个表单定义单一模式并传递到表单级别，错误会自动传播到各字段。

### 标准模式库

TanStack Form 原生支持所有遵循 [Standard Schema 规范](https://github.com/standard-schema/standard-schema) 的库，最著名的包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)
- [Effect/Schema](https://effect.website/docs/schema/standard-schema/)

_注意_：请确保使用模式库的最新版本，旧版本可能尚未支持 Standard Schema。

使用这些库的模式时，可以像自定义函数一样将其传递给 `validators` 属性：

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

表单和字段级别的异步验证也同样支持：

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
    return <>{/* ... */}</>
  }}
/>
```

如需对 Standard Schema 验证进行更多控制，可以将其与回调函数结合使用：

```tsx
<form.Field
  name="age"
  asyncDebounceMs={500}
  validators={{
    onChangeAsync: async ({ value, fieldApi }) => {
      const errors = fieldApi.parseValueWithSchema(
        z.number().gte(13, 'You must be 13 to make an account'),
      )
      if (errors) return errors
      // 继续您的验证逻辑
    },
  }}
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

## 阻止无效表单提交

`onChange`、`onBlur` 等回调在表单提交时也会执行，若表单无效则会阻止提交。

表单状态对象包含 `canSubmit` 标志，当任何字段无效且表单已被触碰时该标志为 false（即使某些字段根据其 `onChange`/`onBlur` 属性"技术上"无效，`canSubmit` 在表单未被触碰前仍为 true）。

可通过 `form.Subscribe` 订阅该值，例如用于在表单无效时禁用提交按钮（实践中禁用按钮不够友好，建议使用 `aria-disabled`）。

```tsx
const form = useForm(/* ... */)

return (
  /* ... */

  // 动态提交按钮
  <form.Subscribe
    selector={(state) => [state.canSubmit, state.isSubmitting]}
    children={([canSubmit, isSubmitting]) => (
      <button type="submit" disabled={!canSubmit}>
        {isSubmitting ? '...' : 'Submit'}
      </button>
    )}
  />
)
```
