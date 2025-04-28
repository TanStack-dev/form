---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-25T20:33:55.670Z'
id: ui-libraries
title: UI 函式庫
---

## 搭配 UI 函式庫使用 TanStack Form

TanStack Form 是一個無頭 (headless) 函式庫，提供完整的樣式自訂彈性。它能與多種 UI 函式庫相容，包括 `Tailwind`、`Material UI`、`Mantine`，甚至是純 CSS。

本指南以 `Material UI` 和 `Mantine` 為重點，但這些概念同樣適用於您選擇的任何 UI 函式庫。

### 必要條件

在整合 TanStack Form 與 UI 函式庫前，請確保專案中已安裝必要的相依套件：

- 使用 `Material UI` 時，請遵循其[官方網站](https://mui.com/material-ui/getting-started/)的安裝說明
- 使用 `Mantine` 時，請參考其[文件](https://mantine.dev/)

注意：雖然可以混用不同函式庫，但建議保持一致性以維持程式碼簡潔。

### Mantine 整合範例

以下示範如何將 TanStack Form 與 Mantine 元件整合：

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
      // Handle form submission
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

- 首先使用 TanStack 的 `useForm` 鉤子並解構所需屬性。此步驟為選用，您也可以直接使用 `const form = useForm()`。TypeScript 的型別推論能確保兩種寫法都有良好的開發體驗。
- 從 `useForm` 取得的 `Field` 元件接受多種屬性，例如 `validators`。本範例主要聚焦兩個核心屬性：`name` 和 `children`。
  - `name` 屬性用於識別每個 `Field`，例如範例中的 `firstName`
  - `children` 屬性採用渲染屬性 (render props) 模式，讓我們能直接整合元件而無需額外抽象層
- TanStack 的設計大量運用渲染屬性，在 `Field` 元件中提供完整的型別安全。當與 Mantine 元件 (如 `TextInput`) 整合時，我們選擇性地解構 `state.value`、`handleChange` 和 `handleBlur` 等屬性。這種選擇性解構是因為 `TextInput` 與 children 中取得的 `field` 在型別上有些微差異。
- 遵循這些步驟，您就能無縫整合 Mantine 元件與 TanStack Form
- 此方法同樣適用於其他元件如 `Checkbox`，確保不同 UI 元素間的整合一致性

### 搭配 Material UI 使用

整合 Material UI 元件的流程類似。以下是使用 Material UI 的 TextField 和 Checkbox 範例：

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

- 整合方式與 Mantine 相同
- 主要差異在於 Material UI 元件的特定屬性和樣式選項
