---
source-updated-at: '2025-03-14T21:54:05.000Z'
translation-updated-at: '2025-04-12T04:12:06.653Z'
id: form-validation
title: 表单验证
---

表单与字段验证 (Form and Field Validation)

TanStack Form 的核心功能之一是高度可定制的验证机制：

- 可自由控制验证时机（输入时、修改时、失焦时、提交时等）
- 验证规则既可在字段级别定义，也可在表单级别定义
- 支持同步验证和异步验证（例如通过 API 调用验证）

## 验证触发时机

完全由您掌控！`<Field />` 组件接受诸如 `onChange` 或 `onBlur` 等回调函数作为 props。这些回调会接收字段当前值和 fieldAPI 对象，您可据此执行验证。若发现验证错误，只需返回字符串类型的错误信息，该信息将通过 `field().state.meta.errors` 暴露。

示例代码：

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
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {field().state.meta.errors ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

上例实现了每次按键时的验证（`onChange`）。若需改为失焦时验证，代码调整如下：

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
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        // 监听字段的失焦事件
        onBlur={field().handleBlur}
        // 仍需实现 onInput 以保证 TanStack Form 能接收变更
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {field().state.meta.errors ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

通过实现不同的回调函数，您可以精确控制验证时机。甚至可以在不同时机执行不同验证：

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
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        // 监听字段的失焦事件
        onBlur={field().handleBlur}
        // 仍需实现 onInput 以保证 TanStack Form 能接收变更
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {field().state.meta.errors ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

上例展示了同一字段在不同时机（按键时和失焦时）的差异化验证。由于 `field().state.meta.errors` 是数组结构，所有当前相关错误都会显示。您也可使用 `field().state.meta.errorMap` 按验证时机（onChange、onBlur 等）获取错误信息。更多错误展示方案见下文。

## 错误信息展示

验证生效后，可通过以下方式展示错误信息：

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
        {field().state.meta.errors.length ? (
          <em>{field().state.meta.errors.join(',')}</em>
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
      {field().state.meta.errorMap['onChange'] ? (
        <em>{field().state.meta.errorMap['onChange']}</em>
      ) : null}
    </>
  )}
</form.Field>
```

需注意 `errors` 数组和 `errorMap` 的类型与验证器返回类型一致：

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
      {!field().state.meta.errorMap['onChange']?.isOldEnough ? (
        <em>The user is not old enough</em>
      ) : null}
    </>
  )}
</form.Field>
```

## 字段级验证 vs 表单级验证

如前所示，每个 `<Field>` 可通过 `onChange`、`onBlur` 等回调定义专属验证规则。也可通过向 `createForm()` 钩子传递类似回调，在表单级别（而非逐个字段）定义验证规则：

```tsx
export default function App() {
  const form = createForm(() => ({
    defaultValues: {
      age: 0,
    },
    onSubmit: async ({ value }) => {
      console.log(value)
    },
    validators: {
      // 表单级验证器的添加方式与字段级相同
      onChange({ value }) {
        if (value.age < 13) {
          return 'Must be 13 or older to sign'
        }
        return undefined
      },
    },
  }))

  // 订阅表单的 errorMap 以保证更新时重新渲染
  // 也可使用 `form.Subscribe`
  const formErrorMap = form.useStore((state) => state.errorMap)

  return (
    <div>
      {/* ... */}
      {formErrorMap().onChange ? (
        <div>
          <em>There was an error on the form: {formErrorMap().onChange}</em>
        </div>
      ) : null}
      {/* ... */}
    </div>
  )
}
```

## 异步函数验证

虽然多数验证是同步的，但网络请求等异步操作也常被用于验证场景。

为此我们提供了专用的 `onChangeAsync`、`onBlurAsync` 等方法：

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
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {field().state.meta.errors ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

同步与异步验证可共存。例如同一字段可同时定义 `onBlur` 和 `onBlurAsync`：

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
      <label for={field().name}>Age:</label>
      <input
        id={field().name}
        name={field().name}
        value={field().state.value}
        type="number"
        onBlur={field().handleBlur}
        onInput={(e) => field().handleChange(e.target.valueAsNumber)}
      />
      {field().state.meta.errors ? (
        <em role="alert">{field().state.meta.errors.join(', ')}</em>
      ) : null}
    </>
  )}
</form.Field>
```

同步验证方法（`onBlur`）会优先执行，仅当其通过时才会触发异步方法（`onBlurAsync`）。如需改变此行为，可将 `asyncAlways` 选项设为 `true`，此时无论同步方法结果如何都会执行异步验证。

### 内置防抖机制

虽然异步验证适合数据库校验，但每次按键都发起网络请求会导致数据库遭受 DDoS 攻击风险。

我们通过单一属性即可实现异步调用的防抖：

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

这将为所有异步调用添加 500 毫秒防抖延迟。还可针对特定验证属性单独设置：

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

> 此时 `onChangeAsync` 每 1500 毫秒执行一次，而 `onBlurAsync` 仍保持 500 毫秒间隔。

## 通过模式库验证

虽然函数验证更灵活，但代码较冗长。为此可使用模式库 (schema libraries) 实现简洁且类型严格的验证。您还可以为整个表单定义单一模式，错误将自动传播到各字段。

### 标准模式库

TanStack Form 原生支持所有符合 [Standard Schema 规范](https://github.com/standard-schema/standard-schema) 的库，主要包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_注意：_ 请使用模式库的最新版本，旧版本可能尚未支持 Standard Schema。

使用这些库的模式时，可像自定义函数一样传递给 `validators` props：

```tsx
import { z } from 'zod'

// ...

const form = createForm(() => ({
  // ...
}))

;<form.Field
  name="age"
  validators={{
    onChange: z.number().gte(13, 'You must be 13 to make an account'),
  }}
  children={(field) => {
    return <>{/* ... */}</>
  }}
/>
```

表单和字段级别的异步验证同样支持：

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

## 阻止无效表单提交

表单提交时也会触发 `onChange`、`onBlur` 等回调，若表单无效则会阻止提交。

表单状态对象包含 `canSubmit` 标志：当任何字段无效且表单已被触碰时该值为 false（即使某些字段"技术上"因 `onChange`/`onBlur` props 而无效，只要表单未被触碰，`canSubmit` 仍为 true）。

可通过 `form.Subscribe` 订阅该值，例如用于在表单无效时禁用提交按钮（实践中应使用 `aria-disabled` 而非禁用按钮以保证可访问性）：

```tsx
const form = createForm(() => ({
  /* ... */
}))

return (
  /* ... */

  // 动态提交按钮
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
  />
)
```
