---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:27:32.107Z'
id: basic-concepts
title: 基本概念
---

# 基本概念と用語

このページでは、`@tanstack/lit-form` ライブラリで使用される基本概念と用語を紹介します。これらの概念を理解することで、ライブラリとそのLitとの連携についてより深く理解し、効果的に活用できるようになります。

## フォームオプション

`formOptions` 関数を使用して、複数のフォーム間で共有可能なフォームオプションを作成できます。

例:

```tsx
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    employed: false,
    jobTitle: '',
  } as Employee,
})
```

## フォームインスタンス

フォームインスタンスは、個々のフォームを表現するオブジェクトで、フォームを操作するためのメソッドやプロパティを提供します。`@tanstack/lit-form` が提供する `TanStackFormController` インターフェースを使用してフォームインスタンスを作成します。`TanStackFormController` は現在のフォームのクラス (`this`) とデフォルトのフォームオプションでインスタンス化され、フォーム状態の初期化、フォーム送信の処理、フォームフィールドとそのバリデーションを管理するメソッドを提供します。

```tsx
#form = new TanStackFormController(this, {
  defaultValues: {
    firstName: '',
    lastName: '',
    employed: false,
    jobTitle: '',
  } as Employee,
})
```

`formOptions` を使用せずにスタンドアロンの `TanStackFormController` API を使用してフォームインスタンスを作成することもできます:

```tsx
#form = new TanStackFormController(this, {
  ...formOpts,
})
```

## フィールド

フィールドは、テキスト入力やチェックボックスなどの単一のフォーム入力要素を表します。フィールドは、フォームインスタンスが提供する `field(FieldOptions, callback)` を使用して作成されます。このコンポーネントは `FieldOptions` オブジェクトと、`FieldApi` オブジェクトを受け取るコールバック関数を受け入れます。このオブジェクトは、フィールドの現在の値を取得するメソッド、入力変更を処理するメソッド、およびブラーイベントを処理するメソッドを提供します。

例:

```ts
 ${this.#form.field(
    {
      name: `firstName`,
      validators: {
        onChange: ({ value }) =>
          value.length < 3 ? "Not long enough" : undefined,
        },
      },
      (field: FieldApi<Employee, "firstName">) => {
        return html` <div>
          <label class="first-name-label">First Name</label>
          <input
           id="firstName"
           type="text"
           class="first-name-input"
           placeholder="First Name"
           @blur="${() => field.handleBlur()}"
           .value="${field.state.value}"
           @input="${(event: InputEvent) => {
           if (event.currentTarget) {
            const newValue = (event.currentTarget as HTMLInputElement).value;
            field.handleChange(newValue);
           }
          }}"
        />
      </div>`;
    },
)}
```

## フィールド状態

各フィールドには、現在の値、バリデーションステータス、エラーメッセージ、およびその他のメタデータを含む独自の状態があります。フィールドの状態には `field.state` プロパティを使用してアクセスできます。

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

ユーザーがフィールドとどのようにインタラクションしているかを確認するのに非常に役立つ3つのフィールド状態があります。フィールドは、ユーザーがクリック/タブでフォーカスを当てると「タッチされた (touched)」状態になり、ユーザーが値を変更するまで「未変更 (pristine)」状態のまま、値が変更されると「変更済み (dirty)」状態になります。これらの状態は、以下のように `isTouched`、`isPristine`、`isDirty` フラグで確認できます。

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![フィールド状態](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)
