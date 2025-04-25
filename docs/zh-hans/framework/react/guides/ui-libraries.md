---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-12T04:09:36.714Z'
id: ui-libraries
title: UI库集成
---
## 在 UI 库中使用 TanStack Form

TanStack Form 是一个无头 (headless) 库，为您提供完全的样式定制灵活性。它能与多种 UI 库兼容，包括 `Tailwind`、`Material UI`、`Mantine` 甚至原生 CSS。

本指南主要关注 `Material UI` 和 `Mantine`，但相关概念同样适用于您选择的任何 UI 库。

### 准备工作

在集成 TanStack Form 与 UI 库之前，请确保项目中已安装必要的依赖项：

- 对于 `Material UI`，请按照其[官网](https://mui.com/material-ui/getting-started/)的安装说明操作
- 对于 `Mantine`，请参考其[文档](https://mantine.dev/)

注意：虽然可以混合使用不同库，但通常建议坚持使用一个库以保持一致性并减少冗余。

### Mantine 集成示例

以下示例演示了如何将 TanStack Form 与 Mantine 组件集成：

```tsx
import { TextInput, Checkbox } from '@mantine/core'
import { useForm } from '@tanstack/react-form'

export default function App() {
  const { Field, handleSubmit, state } = useForm({
    defaultValues: {
      firstName: '',
      lastName: '',
      isChecked: false,
    },
    onSubmit: async ({ value }) => {
      // 处理表单提交
      console.log(value)
    },
  })

  return (
    <>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          handleSubmit()
        }}
      >
        <Field
          name="firstName"
          children={({ state, handleChange, handleBlur }) => (
            <TextInput
              defaultValue={state.value}
              onChange={(e) => handleChange(e.target.value)}
              onBlur={handleBlur}
              placeholder="Enter your name"
            />
          )}
        />
        <Field
          name="isChecked"
          children={({ state, handleChange, handleBlur }) => (
            <Checkbox
              onChange={(e) => handleChange(e.target.checked)}
              onBlur={handleBlur}
              checked={state.value}
            />
          )}
        />
      </form>
      <div>
        <pre>{JSON.stringify(state.values, null, 2)}</pre>
      </div>
    </>
  )
}
```

- 首先我们使用 TanStack 的 `useForm` 钩子并解构所需属性。此步骤是可选的，您也可以选择使用 `const form = useForm()`。无论采用哪种方式，TypeScript 的类型推断都能确保良好的开发体验
- 从 `useForm` 获取的 `Field` 组件接受多个属性，例如 `validators`。本示例主要关注两个核心属性：`name` 和 `children`
  - `name` 属性用于标识每个 `Field`，例如示例中的 `firstName`
  - `children` 属性运用了渲染属性 (render props) 的概念，让我们无需额外抽象就能集成组件
- TanStack 的设计大量使用渲染属性，在 `Field` 组件内部提供对 `children` 的访问。这种方式是完全类型安全的。当与 Mantine 组件（如 `TextInput`）集成时，我们选择性地解构 `state.value`、`handleChange` 和 `handleBlur` 等属性。这种选择性处理是因为 `TextInput` 与我们从 children 获取的 `field` 在类型上存在细微差异
- 通过以上步骤，您可以无缝地将 Mantine 组件与 TanStack Form 集成
- 此方法同样适用于其他组件（如 `Checkbox`），确保不同 UI 元素间的集成一致性

### Material UI 集成示例

集成 Material UI 组件的流程类似。以下是使用 Material UI 的 TextField 和 Checkbox 的示例：

```tsx
        <Field
            name="lastName"
            children={({ state, handleChange, handleBlur }) => {
              return (
                <TextField
                  id="filled-basic"
                  label="Filled"
                  variant="filled"
                  defaultValue={state.value}
                  onChange={(e) => handleChange(e.target.value)}
                  onBlur={handleBlur}
                  placeholder="Enter your last name"
                />
              );
            }}
          />

           <Field
            name="isMuiCheckBox"
            children={({ state, handleChange, handleBlur }) => {
              return (
                <MuiCheckbox
                  onChange={(e) => handleChange(e.target.checked)}
                  onBlur={handleBlur}
                  checked={state.value}
                />
              );
            }}
          />
```

- 集成方式与 Mantine 相同
- 主要区别在于 Material UI 组件的特定属性和样式选项
