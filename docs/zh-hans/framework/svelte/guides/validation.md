---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-29T23:16:05.501Z'
id: form-validation
title: 表单验证
---

表单与字段验证 (Form and Field Validation)

TanStack Form 的核心功能之一是验证机制。该库提供了高度可定制的验证方案：

- 可自由控制验证触发时机（值变更时、输入时、失焦时、提交时等）
- 验证规则可在字段级别或表单级别定义
- 支持同步验证和异步验证（例如通过 API 调用返回结果）

## 验证触发时机

完全由您掌控！`<form.Field />` 组件接受 `onChange` 或 `onBlur` 等回调函数作为属性。这些回调会接收字段当前值和 fieldAPI 对象，便于执行验证逻辑。若发现验证错误，只需返回字符串类型的错误信息，该信息将存储在 `field.state.meta.errors` 中。

示例代码：

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

上例实现了每次输入时触发验证（`onchange`）。若需改为失焦时验证，代码调整如下：

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

通过实现不同的回调函数，您可以精确控制验证时机。甚至可以在不同时机执行不同验证逻辑：

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

上例展示了同一字段在不同时机（输入时和失焦时）执行不同验证。由于 `field.state.meta.errors` 是数组类型，会显示当前所有相关错误。也可以通过 `field.state.meta.errorMap` 获取基于验证时机的错误（onChange、onBlur 等）。下文将详细介绍错误展示。

## 错误展示

配置验证后，可将错误数组映射到界面展示：

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

或使用 `errorMap` 属性获取特定错误：

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

需注意 `errors` 数组和 `errorMap` 的类型与验证器返回类型一致：

```svelte
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 13 ? { isOldEnough: false } : undefined),
  }}
>
  {#snippet children(field)}
    <!-- ... -->
    <!-- errorMap.onChange 类型为 `{isOldEnough: false} | undefined` -->
    <!-- meta.errors 类型为 `Array<{isOldEnough: false} | undefined>` -->
    {#if field.state.meta.errorMap['onChange']?.isOldEnough}
        <em>The user is not old enough</em>
    {/if}
  {/snippet}
</form.Field>
```

## 字段级与表单级验证

如前所示，每个 `<form.Field>` 可通过 `onChange`、`onBlur` 等回调定义独立验证规则。也可以通过 `createForm()` 钩子在表单级别定义验证规则：

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
      // 表单级验证器的添加方式与字段级相同
      onChange({ value }) {
        if (value.age < 13) {
          return 'Must be 13 or older to sign'
        }
        return undefined
      },
    },
  }))

  // 订阅表单错误映射以触发渲染
  // 也可使用 `form.Subscribe`
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

## 异步函数验证

虽然大多数验证是同步的，但很多场景需要通过网络请求等异步操作进行验证。

为此，我们提供了专用的 `onChangeAsync`、`onBlurAsync` 等方法：

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

同步与异步验证可以共存。例如同一字段可同时定义 `onBlur` 和 `onBlurAsync`：

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

同步验证方法（`onBlur`）会优先执行，只有同步验证通过后才会执行异步方法（`onBlurAsync`）。如需改变此行为，可将 `asyncAlways` 选项设为 `true`，这样无论同步方法结果如何都会执行异步方法。

### 内置防抖机制

虽然异步验证适合数据库校验，但每次输入都发起网络请求可能导致数据库遭受 DDoS 攻击。

我们通过简单属性即可实现异步调用的防抖：

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

这将为所有异步调用添加 500 毫秒防抖延迟。还可以针对特定验证属性单独设置：

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

> 这将使 `onChangeAsync` 每 1500 毫秒执行一次，而 `onBlurAsync` 仍保持 500 毫秒间隔。

## 通过模式库验证

虽然函数验证更灵活，但可能显得冗长。为此，有些库提供基于模式 (schema) 的验证方案，能大幅简化类型严格的验证流程。您还可以为整个表单定义单一模式，并将其传递到表单级别，错误会自动传播到各个字段。

### 标准模式库

TanStack Form 原生支持所有符合 [Standard Schema 规范](https://github.com/standard-schema/standard-schema) 的库，最典型的有：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_注意：_ 请确保使用模式库的最新版本，旧版本可能尚未支持 Standard Schema。

使用这些库的模式时，可以像自定义函数一样传递给 `validators` 属性：

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

表单和字段级别的异步验证同样支持：

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

## 阻止无效表单提交

表单提交时也会运行 `onChange`、`onBlur` 等回调，如果表单无效则会阻止提交。

表单状态对象包含 `canSubmit` 标志，当任何字段无效且表单已被触碰时该标志为 false（即使某些字段根据其 `onChange`/`onBlur` 属性"技术上"无效，只要表单未被触碰，`canSubmit` 仍为 true）。

可通过 `form.Subscribe` 订阅该值，例如用于在表单无效时禁用提交按钮（实践中禁用按钮不利于可访问性，建议使用 `aria-disabled`）。

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    /* ... */
  }))
</script>

<!-- ... -->

<!-- 动态提交按钮 -->
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
