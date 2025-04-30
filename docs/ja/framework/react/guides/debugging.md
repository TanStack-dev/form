---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:25:29.015Z'
id: debugging
title: デバッグ
---

以下はコンソールで表示される可能性のある一般的なエラーとその修正方法のリストです。

## 非制御 (uncontrolled) 入力が制御 (controlled) 入力に変更される場合

コンソールに以下のエラーが表示される場合:

```
Warning: A component is changing an uncontrolled input to be controlled. This is likely caused by the value changing from undefined to a defined value, which should not happen. Decide between using a controlled or uncontrolled input element for the lifetime of the component. More info: https://reactjs.org/link/controlled-components
```

これはおそらく、`useForm` フックまたは `form.Field` コンポーネントの使用時に `defaultValues` を指定し忘れていることが原因です。このエラーは、フォームの値が初期化される前に入力がレンダリングされ、テキスト入力が行われると値が `undefined` から `""` に変更される際に発生します。

## フィールド値の型が `unknown` になる場合

`form.Field` を使用していて、`field.state.value` の値を確認した際にフィールド値の型が `unknown` になっている場合、フォームの型が大きすぎて安全に評価できない可能性があります。

これは通常、フォームをより小さなフォームに分割するか、フォームに対してより具体的な型を使用すべきであることを示しています。

この問題の回避策として、TypeScript の `as` キーワードを使用して `field.state.value` をキャストすることができます:

```tsx
const value = field.state.value as string
```

## `Type instantiation is excessively deep and possibly infinite`

`tsc` を実行した際にコンソールに以下のエラーが表示される場合:

```
Type instantiation is excessively deep and possibly infinite
```

これは、型定義で検出できなかったバグに遭遇したことを意味します。私たちは型が可能な限り正確になるように最善を尽くしましたが、TypeScript が型の複雑さに対処しきれないいくつかのエッジケースが存在します。

この問題を修正できるよう、[GitHub で問題を報告](https://github.com/TanStack/form/issues)してください。その際、デバッグに役立つ最小限の再現コードを含めるようにしてください。

> このエラーは TypeScript のエラーであり、ランタイムエラーではないことに注意してください。つまり、ユーザーのマシン上ではコードは期待通りに実行されます。
