---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-12T04:09:26.089Z'
id: debugging
title: 调试
---
以下是您可能在控制台中遇到的常见错误列表及其解决方法。

## 将非受控输入更改为受控输入

如果在控制台中看到以下错误：

```
警告：组件正在将非受控输入更改为受控输入。这可能是由于值从 undefined 变为已定义值导致的，这种情况不应发生。请为组件的整个生命周期选择使用受控或非受控输入元素。更多信息：https://reactjs.org/link/controlled-components
```

很可能是因为您在使用 `useForm` Hook 或 `form.Field` 组件时忘记了设置 `defaultValues`。该错误的发生是因为输入框在表单值初始化之前就被渲染，导致文本输入时值从 `undefined` 变为 `""`。

## 字段值的类型为 `unknown`

如果您使用 `form.Field` 并检查 `field.state.value` 时，发现字段值的类型为 `unknown`，这可能是因为表单的类型过大，导致我们无法安全推断类型。

这通常意味着您应该将表单拆分为更小的表单，或者为表单使用更具体的类型。

临时解决方案是使用 TypeScript 的 `as` 关键字对 `field.state.value` 进行类型断言：

```tsx
const value = field.state.value as string
```

## `类型实例化过深且可能无限`

如果在运行 `tsc` 时在控制台中看到以下错误：

```
类型实例化过深且可能无限
```

您遇到了我们在类型定义中未捕获的 bug。尽管我们已尽力确保类型的准确性，但在某些边缘情况下，TypeScript 仍难以处理我们类型的复杂性。

请[在 GitHub 上向我们报告此问题](https://github.com/TanStack/form/issues)，以便我们修复。请确保包含一个最小复现示例，以便我们帮助您调试。

> 请注意，此错误是 TypeScript 错误而非运行时错误。这意味着您的代码在用户机器上仍会按预期运行。
