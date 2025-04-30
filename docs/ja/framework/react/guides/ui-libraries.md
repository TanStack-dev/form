---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:25:39.621Z'
id: ui-libraries
title: UIライブラリ
---

## UIライブラリとの連携におけるTanStack Formの使用法

TanStack Formはヘッドレスライブラリであり、自由にスタイリングできる柔軟性を提供します。`Tailwind`、`Material UI`、`Mantine`、あるいはプレーンなCSSなど、さまざまなUIライブラリと互換性があります。

このガイドでは`Material UI`と`Mantine`に焦点を当てますが、概念は任意のUIライブラリに適用可能です。

### 前提条件

TanStack FormをUIライブラリと統合する前に、プロジェクトに必要な依存関係がインストールされていることを確認してください:

- `Material UI`の場合、[公式サイト](https://mui.com/material-ui/getting-started/)のインストール手順に従ってください。
- `Mantine`の場合、[ドキュメント](https://mantine.dev/)を参照してください。

注: 複数のライブラリを混在させることも可能ですが、一貫性を保ち肥大化を防ぐため、1つのライブラリに統一することを推奨します。

### Mantineとの連携例

以下はTanStack FormとMantineコンポーネントを統合する例です:

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

- 最初にTanStackの`useForm`フックを使用し、必要なプロパティを分割代入します。この手順はオプションで、`const form = useForm()`のように使用することも可能です。TypeScriptの型推論により、どちらのアプローチでもスムーズな開発体験が得られます。
- `useForm`から得られる`Field`コンポーネントは`validators`など複数のプロパティを受け取りますが、このデモでは主に`name`と`children`の2つのプロパティに焦点を当てます。
  - `name`プロパティは各`Field`を識別します（例では`firstName`）。
  - `children`プロパティはレンダープロップの概念を活用し、不必要な抽象化なしにコンポーネントを統合できます。
- TanStackの設計はレンダープロップを多用しており、`Field`コンポーネント内で`children`にアクセスできます。このアプローチは完全にタイプセーフです。`TextInput`などのMantineコンポーネントと統合する際、`state.value`、`handleChange`、`handleBlur`などのプロパティを選択的に分割代入します。この選択的なアプローチは、`TextInput`と`children`で得られる`field`の型の微妙な違いによるものです。
- これらの手順に従うことで、MantineコンポーネントとTanStack Formをシームレスに統合できます。
- この方法論は`Checkbox`などの他のコンポーネントにも同様に適用可能で、さまざまなUI要素間で一貫した統合を実現します。

### Material UIとの連携

Material UIコンポーネントとの統合プロセスも同様です。以下はMaterial UIのTextFieldとCheckboxを使用した例です:

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

- 統合アプローチはMantineの場合と同じです。
- 主な違いは、Material UIコンポーネント固有のプロパティとスタイリングオプションにあります。
